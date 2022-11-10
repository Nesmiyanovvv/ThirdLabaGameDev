# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #3 выполнил:

+ Несмиянов Матвей Андреевич
+ РИ-210930 Отметка о выполнении заданий (заполняется студентом):

| Задание  | Выполнение | Баллы |
| ------------- | ------------- | ------------- |
| Задание 1 | * | 60 | 
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:

+ к.т.н., доцент Денисов Д.В.
+ к.э.н., доцент Панов М.А.
+ ст. преп., Фадеев В.О.


Структура отчета

+ Данные о работе: название работы, ФИО, группа, выполненные задания.
+ Цель работы.
+ Задание 1.
  + Код реализации выполнения задания. Визуализация результатов выполнения.
+ Задание 2.
  + Код реализации выполнения задания. Визуализация результатов выполнения.
+ Задание 3.
  + Код реализации выполнения задания. Визуализация результатов выполнения.
+ Выводы.

# Цель работы

Познакомиться с программными средствами для создания системы машинного обучения и ее интеграции в Unity.

# Задание 1. Реализовать систему машинного обучения в связке Python - Google-Sheets – Unity.

Создание проекта Unity и добавление в него пакетов ML Agent.

<img width="362" alt="197768538-a54aa5d9-f31f-4619-a309-9be94ad0c9fc" src="https://user-images.githubusercontent.com/94855641/201148905-81d1351a-f1e0-4cac-a0d4-2aa3a716b371.png">

Серия команд для создания и активации нового ML-агента, а также для скачивания необходимых библиотек:

``conda create -n MLAgent python=3.6``

``conda activate MLAgent``

``pip install mlagents``

``pip install torch==1.7.1+cu110 --extra-index-url https://download.pytorch.org/whl/cu110``

Создание на сцене плоскости, куба и сферы
<img width="1329" alt="197770689-0ede7400-373a-4048-9a7e-5e869723b76b" src="https://user-images.githubusercontent.com/94855641/201150158-0f46afd2-4bf5-451d-b25e-d02aba085551.png">

Добавление приведенного ниже скрипта в RollerAgent.cs
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public override void OnEpisodeBegin()
    {
        if (transform.localPosition.y < 0)
        {
            rBody.angularVelocity = Vector3.zero;
            rBody.velocity = Vector3.zero;
            transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }
    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.localPosition);

        if(distanceToTarget < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (transform.localPosition.y < 0)
        {
            EndEpisode();
        }
    }
}
```
Добавление объекту «сфера» компонентов Rigidbody, Decision Requester, Behavior Parameters и их настройка

<img width="410" alt="197774831-562d6780-4654-4361-be3e-7eeae8276194" src="https://user-images.githubusercontent.com/94855641/201150573-3b7a9d74-461d-49a0-bbc6-92ee21e8d5a0.png">

Запуск агента

<img width="871" alt="197776257-2beb2f11-68c2-4b9f-ad5a-be8dbcbab767" src="https://user-images.githubusercontent.com/94855641/201150694-ed59f09e-ac52-4eaf-bdd2-74e2f410070b.png">


Видео запуска сцены с 27 моделями «Плоскость-Сфера-Куб»


https://user-images.githubusercontent.com/94855641/201153684-648c9107-4326-410f-9a24-a37c19436b74.mov

Видео проверки работы обученной модели


https://user-images.githubusercontent.com/94855641/201154651-e74b9108-75f8-428c-aeb8-f308501ba6b4.mov



# Выводы

Чем больше моделей обучается одновременно, тем быстрее будет виден результат обучения

# Задание 2
Описание строк файла конфигурации нейронной сети

| Параметр | Описание |
| ------------- | ------------- |
| `trainer_type` | Тип тренажера для использования, по умолчанию обучение с наградой. |
| `batch_size` | Количество опытов на каждой итерации градиентного спуска. Всегда должно быть в несколько раз меньше, чем `buffer_size`. Если используются непрерывные действия, это значение должно быть большим (порядка 1000). Если используются только дискретные действия, это значение должно быть меньше (порядка 10 секунд). |
| `buffer_size` | Количество опыта, который необходимо собрать перед обновлением модели политики. Соответствует тому, сколько опыта должно быть собрано, прежде чем мы приступим к какому-либо изучению или обновлению модели. Должно быть в несколько раз больше, чем `batch_size`. Обычно больший размер буфера соответствует более стабильным обновлениям обучения |
| `learning_rate` | Начальная скорость обучения для градиентного спуска. Соответствует силе каждого шага обновления градиентного спуска. Обычно это значение следует уменьшить, если тренировка нестабильна, а вознаграждение постоянно не увеличивается. |
| `beta` | Регулировка энтропии, которая делает политику "более случайной". Это гарантирует, что агенты должным образом исследуют пространство действий во время обучения. Увеличение этого параметра обеспечит выполнение большего количества случайных действий. |
| `epsilon` | Влияет на то, насколько быстро политика может развиваться во время обучения. Соответствует допустимому порогу расхождения между старой и новой политиками при обновлении с градиентным спуском. Установка малого значения приведет к более стабильным обновлениям, но также замедлит процесс обучения. |
| `lambd` | Параметр регуляризации, используется при расчете обобщенной оценки преимущества GAE. Это можно рассматривать как то, насколько агент полагается на свою текущую оценку стоимости при расчете обновленной оценки стоимости. Низкие значения соответствуют тому, что агент больше полагается на текущую оценку стоимости, а высокие значения соответствуют тому, что агент больше полагается на фактические вознаграждения, полученные в среде. |
| `num_epoch` | Количество проходов через буфер опыта при выполнении оптимизации градиентного спуска. Чем больше размер пакета, тем больше будет проходов. Уменьшение этого параметра обеспечит более стабильные обновления за счет более медленного обучения. |
| `learning_rate_schedule` | Определяет, как скорость обучения меняется с течением времени. Для `PPO` рекомендуется снижать скорость обучения до `max_steps`, чтобы обучение проходило более стабильно. |
| `normalize` | Применяется ли нормализация к входным данным векторного наблюдения. Эта нормализация основана на текущем среднем значении и дисперсии векторного наблюдения. Нормализация может быть полезна в случаях со сложными проблемами непрерывного управления, но может быть вредной при решении более простых задач дискретного управления. |
| `hidden_units` | Количество узлов в скрытых слоях нейронной сети. Соответствует количеству узлов в каждом полностью связном слое нейронной сети. Для простых задач, где правильное действие представляет собой простую комбинацию входных данных наблюдения, параемтр должен принимать небольшое значение. Для задач, где действие представляет собой очень сложное взаимодействие между переменными наблюдения, параемтр должен принимать значение больше. |
| `num_layers` | Количество скрытых слоев в нейронной сети. Соответствует тому, сколько скрытых слоев присутствует после ввода наблюдения или после CNN кодирования визуального наблюдения. Для простых задач меньшее количество слоев, скорее всего, будет обучаться быстрее и эффективнее. Для более сложных задач управления может потребоваться больше слоев. |
| `gamma` | Коэффициент дисконтирования для будущих вознаграждений, поступающих от окружения. Это можно рассматривать как то, насколько далеко в будущем агент должен заботиться о возможных вознаграждениях. В ситуациях, когда агент должен действовать в настоящем, чтобы подготовиться к вознаграждению в будущем, это значение должно быть большим. В тех случаях, когда вознаграждение приходит быстрее, оно может быть меньше. Должно быть строго меньше 1. |
| `strength` | Коэффициент, на который можно умножить вознаграждение, получаемое от окружения. Обычно диапазоны варьируются в зависимости от сигнала вознаграждения. |
| `max_steps` | Общее количество шагов, которые должны быть выполнены в среде (или во всех средах, если используется несколько параллельно) до завершения процесса обучения. Если есть несколько агентов с одинаковым именем в среде, все шаги, выполняемые этими агентами, будут способствовать одинаковому количеству `max_steps`. |
| `time_horizon` | Сколько шагов опыта нужно собрать каждому агенту, прежде чем добавить его в буфер опыта. Когда этот предел достигается до окончания эпизода, оценка стоимости используется для прогнозирования общего ожидаемого вознаграждения от текущего состояния агента. Таким образом, этот параметр балансирует между менее предвзятой, но более высокой оценкой дисперсии (длительный временной горизонт) и более предвзятой, но менее разнообразной оценкой (короткий временной горизонт). В тех случаях, когда в эпизоде часто бывают награды или эпизоды непомерно велики, меньшее количество может быть более идеальным. Это число должно быть достаточно большим, чтобы охватить все важное поведение в последовательности действий агента. |
| `summary_freq` | Количество опыта, который необходимо собрать перед созданием и отображением статистики обучения. Определяет степень детализации графиков в Tensorboard. |

Информация о компонентах

| Компонента | Описание |
| ------------- | ------------- |
| `Decision  Requester` | При обучении с подкреплением обученная система сначала должна принять решение, прежде чем она сможет предпринять действие. Целью `Decision Requester` является извлечение и отправка вычисленного числового значения, которое формирует ответ в данный момент времени. По отношению к агентам ML это совокупность буферов дискретных и непрерывных действий. Вызывается через определенные промежутки времени, которые можно задать с помощью полосы прокрутки периода принятия решения. |
| `Behavior  Parameters` | Определяет, как агент принимает решения. |
| `Behavior  Parameters ->  Vector  Observation  Space`| Прежде чем принять решение, агент собирает свои наблюдения о своем положении в мире. Векторное наблюдение - это вектор чисел с плавающей запятой, которые содержат релевантную информацию для принятия агентом решений. Если в параметрах указано `Space Size = 8`, это означает, что вектор признаков, содержащий наблюдения агента, содержит восемь элементов: компоненты x и z вращения куба агента и компоненты x, y и z относительного положения и скорости шара. |
| `Behavior  Parameters ->  Actions` | Агенту даются инструкции в форме действий. ML-Agents Toolkit классифицирует действия на два типа: непрерывные и дискретные. |

# Задание 3. Доработка сцены и обучение ML-Agent таким образом, чтобы шар перемещался между двумя кубами разного цвета.

Для начала был создан второй куб другого цвета.

Затем доработан скрипт на C#

```
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    
    public GameObject FirstTarget;
    public GameObject SecondTarget;
    public float forceMultiplier = 10;
    
    private bool _canPickFirst = true;
    private bool _canPickSecond = true;
    
    void Start() => rBody = GetComponent<Rigidbody>();

    public override void OnEpisodeBegin()
    {
        if (transform.localPosition.y < 0)
        {
            rBody.angularVelocity = Vector3.zero;
            rBody.velocity = Vector3.zero;
            transform.localPosition = new Vector3(0, 0.5f, 0);
        }
        
        _canPickFirst = _canPickSecond = true;
        FirstTarget.SetActive(true);
        SecondTarget.SetActive(true);
        FirstTarget.transform.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
        SecondTarget.transform.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(FirstTarget.transform.localPosition);
        sensor.AddObservation(SecondTarget.transform.localPosition);
        sensor.AddObservation(_canPickFirst);
        sensor.AddObservation(_canPickSecond);
        sensor.AddObservation(transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }

    private bool CheckPicking(bool targetState, Transform target)
    {
        return targetState && Vector3.Distance(transform.localPosition, target.localPosition) < 1.42f;
    }
    
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        var controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        if (transform.localPosition.y < 0) 
            EndEpisode();
        
        if (CheckPicking(_canPickFirst, FirstTarget.transform))
        {
            FirstTarget.SetActive(false);
            _canPickFirst = false;
            
        }
        if (CheckPicking(_canPickSecond, SecondTarget.transform))
        {
            SecondTarget.SetActive(false);
            _canPickSecond = false;
        }

        if (!_canPickFirst && !_canPickSecond)
        {
            SetReward(1.0f);
            EndEpisode();
        }
    }
}
```

Для того, чтобы следить за второй целью, добавлено поле `public GameObject SecondTarget`, которое принимает вторую цель.

Поля `_canPickFirst` и `_canPickSecond` являются флагами, которые означают, можно ли подобрать первую или вторую цель соответственно.

В методе `OnEpisodeBegin()` добавлен сброс приведенных выше значений.

в методе `CollectObservations(VectorSensor sensor)` добавлено отслеживание за новыми сущностями, предварительно изменив значение `Space Size` на 13.

В методе `OnActionReceived(ActionBuffers actionBuffers)` модель получает поощрение, только если шар приблизился к обоим кубам.

Видео работы обученной модели


https://user-images.githubusercontent.com/94855641/201159045-d90f7241-6bbb-4fc9-a53e-139fef89b15e.mov

# Выводы

Баланс в играх - это равенство, равновесие в оружии, тактиках, персонажах и других игровых объектах. Игровой баланс необходим для того, чтобы игроку было интересно играть в игру, она не надоедала однообразием. Например, чередование уровня сложности в различных сценах игры. В многопользовательских играх баланс необходим для того, чтобы все игроки чувствовали "честность" правил и разность сил, например одинаковые условия в начале матча. Пользователь не должен ощущать превосходство со стороны соперника, иначе он просто не будет играть в игру.

Машинное обучение позволяет более эффективно решать проблемы с балансом в играх. Обычно этот процесс отнимает у человека огромное количество времени, однако машины могут помочь ему с этим. Также нередко системы машинного обучения способны более тонко настроить баланс, что впоследствии отразится на вовлеченности пользователя в игровой процесс.
