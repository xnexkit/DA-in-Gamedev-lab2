# DA-in-Gamedev-lab2
# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил:
- Шмаков Никита Владимирович
- ФО210005
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Познакомиться с программными средствами для организции передачи данных между инструментами google, Python и Unity

## Задание 1
### Реализовать совместную работу и передачу данных в связке Python - Google-Sheets – Unity.

- В облачном сервисе google console подключить API для работы с google sheets и google drive.
![GC2 0](https://user-images.githubusercontent.com/45539357/194366778-6117632b-338f-4c43-90a6-fd01d7763ce4.png)

- Реализовать запись данных из скрипта на python в google-таблицу. Данные описывают изменение темпа инфляции на протяжении 11 отсчётных периодов, с учётом стоимости игрового объекта в каждый период.
![GC3](https://user-images.githubusercontent.com/45539357/194367303-a0e1c514-a5b5-4412-990d-f05177f32001.png)

- Создать новый проект на Unity, который будет получать данные из google-таблицы, в которую были записаны данные в предыдущем пункте.
![GC4](https://user-images.githubusercontent.com/45539357/194367703-02c160a6-2444-4e86-83f7-d2206b9dde52.png)

- Написать функционал на Unity, в котором будет воспризводиться аудио-файл в зависимости от значения данных из таблицы.
![GC5](https://user-images.githubusercontent.com/45539357/194368257-98cabf12-a1da-45ac-9e6f-bcb29d2d195d.png)

## Задание 2
### Реализовать запись в Google-таблицу набора данных, полученных с помощью линейной регрессии из лабораторной работы № 1

```python
import numpy as np
import gspread


def model(a, b, x):
    return a * x + b


def loss_function(a, b, x, y):
    num = len(x)
    prediction = model(a, b, x)
    return (0.5 / num) * (np.square(prediction - y)).sum()


def optimize(a, b, x, y):
    num = len(x)
    prediction = model(a, b, x)
    da = (1.0 / num) * ((prediction - y) * x).sum()
    db = (1.0 / num) * ((prediction - y).sum())
    a = a - Lr * da
    b = b - Lr * db
    return a, b


def iterate(a, b, x, y, times):
    for i in range(times):
        a, b = optimize(a, b, x, y)
    return a, b


gc = gspread.service_account(filename="unitydatascience-364713-96b52723f1cc.json")
sh = gc.open("UnitySheets").get_worksheet(1)

x = [3, 21, 22, 34, 54, 34, 55, 67, 89, 99]
x = np.array(x)
y = [2, 22, 24, 65, 79, 82, 55, 130, 150, 199]
y = np.array(y)

a = np.random.rand(1)
print(a)
b = np.random.rand(1)
print(b)
Lr = 0.000001

a,b = iterate(a,b,x,y,1)
prediction = model(a,b,x)
loss = loss_function(a,b,x,y)
print(a,b,loss)

for i in range(len(x)):
    sh.update(('A' + str(i + 1)), str(i + 1))
    sh.update(('B' + str(i + 1)), str(x[i]))
    sh.update(('C' + str(i + 1)), str(prediction[i]))
```
![Task2 1](https://user-images.githubusercontent.com/45539357/194375755-d2f0bdbd-f7c7-45f2-b764-e0cda226d6cf.png)

## Задание 3

### Самостоятельно разработать сценарий воспроизведения звукового сопровождения в Unity в зависимости от изменения считанных данных в задании 2.

- Немного поменял структуру программы и добавил возможность сделать задержку между воспроизведением следующего фрагмента из сходя из данных второй колонки.

```cs
void Update()
{
  if (statusStart || i == dataSet.Count || !dataSet.ContainsKey("Mon_1"))
    return;

  var value = dataSet["Mon_" + i];
  var time = timeStamps["Mon_" + i] / 10;

  switch (value)
  {
    case < 5:
      StartCoroutine(PlaySelectAudio(badSpeak, time));
      break;
    case >= 5 and < 10:
      StartCoroutine(PlaySelectAudio(normalSpeak, time));
      break;
    case >= 10:
      StartCoroutine(PlaySelectAudio(goodSpeak, time));
      break;
  }
}

IEnumerator GoogleSheets()
{
  UnityWebRequest currentResp =
    UnityWebRequest.Get(
      "https://sheets.googleapis.com/v4/spreadsheets/<reference>/values/Лист2?key=<key-code>");

  yield return currentResp.SendWebRequest();

  string rawResp = currentResp.downloadHandler.text;
  var rawJson = JSON.Parse(rawResp);

  foreach (var itemRawJson in rawJson["values"])
  {
    var parseJson = JSON.Parse(itemRawJson.ToString());
    var selectRow = parseJson[0].AsStringList;
    dataSet.Add("Mon_" + selectRow[0], float.Parse(selectRow[2]));
    timeStamps.Add("Mon_" + selectRow[0], int.Parse(selectRow[1]));
  }
}

IEnumerator PlaySelectAudio(AudioClip clip, int time)
{
  statusStart = true;
  selectAudio = GetComponent<AudioSource>();
  selectAudio.clip = clip;
  selectAudio.Play();

  yield return new WaitForSeconds(selectAudio.clip.length + time);

  statusStart = false;
  i++;
  Debug.Log(clip.name);
}
```

![Task3](https://user-images.githubusercontent.com/45539357/194390637-7f04e2d0-eb41-4156-9d6e-2b7f39770b8b.png)

## Выводы

В ходе лабораторной работы я научился работать в Google cloud, добавлять в него новые сервисы и взаимодействовать с ними, используя Python и Unity.

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
