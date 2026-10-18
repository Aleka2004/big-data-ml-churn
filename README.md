# Big Data аналитикасында машиналық оқыту әдістерін қолдану

## Жоба: Customer Churn Prediction with Apache Spark MLlib

Бұл жоба Big Data ортасында машиналық оқыту әдістерін қолдануды көрсетеді. Мақсат — клиенттің қызметтен кету ықтималдығын (`churn`) болжау және үш модельді салыстыру:

- Logistic Regression;
- Random Forest;
- Gradient Boosted Trees (GBT).

Негізгі Big Data нұсқасы **Apache Spark / PySpark MLlib** арқылы орындалады. Репозиторийде Spark жоқ компьютерде жылдам тексеруге арналған `scikit-learn` демо-нұсқасы да бар.

## Жоба құрылымы

```text
big-data-ml-churn/
├── src/
│   ├── train_spark.py       # Негізгі Big Data / Spark MLlib pipeline
│   └── local_demo.py        # Жылдам жергілікті demo және графиктер
├── notebooks/
│   └── big_data_ml_project.ipynb
├── data/
│   └── sample_customers.csv
├── results/
│   ├── metrics.csv
│   ├── model_comparison.png
│   ├── confusion_matrix_best.png
│   └── feature_importance.png
├── requirements.txt
├── publish_to_github.sh
└── README.md
```

## Деректер

Сыртқы жабық деректер қолданылмайды. Код клиенттер туралы синтетикалық деректерді өзі жасайды. Белгілер:

`age`, `tenure_months`, `monthly_spend`, `usage_hours`, `support_calls`, `late_payments`, `satisfaction_score`, `contract_type`, `region`, `has_internet`.

Мақсатты айнымалы: `churn` (0 — клиент қалады, 1 — кетеді).

## 1) Жылдам demo

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python src/local_demo.py
```

Demo 80 000 жазба жасайды, модельдерді үйретеді және `results/` бумасына метрикалар мен графиктерді сақтайды.

Алынған demo нәтижесі (seed=42):

| Модель | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---:|---:|---:|---:|---:|
| Gradient Boosted Trees | 0.7205 | 0.4052 | 0.6962 | 0.5123 | **0.7823** |
| Random Forest | **0.7317** | **0.4146** | 0.6621 | 0.5099 | 0.7787 |
| Logistic Regression | 0.7047 | 0.3898 | **0.7076** | 0.5027 | 0.7774 |

## 2) Big Data нұсқасы — Apache Spark

```bash
pip install pyspark
spark-submit src/train_spark.py --rows 1000000
```

Мысалы, 5 миллион жазбамен:

```bash
spark-submit --driver-memory 4g src/train_spark.py --rows 5000000
```

Pipeline келесі кезеңдерден тұрады:

1. Spark DataFrame ішінде деректерді генерациялау;
2. StringIndexer + OneHotEncoder;
3. VectorAssembler + StandardScaler;
4. train/test бөлу;
5. Logistic Regression, Random Forest, GBT модельдерін үйрету;
6. Accuracy, Precision, Recall, F1 және ROC-AUC бойынша салыстыру.

## Colab

`notebooks/big_data_ml_project.ipynb` файлын Google Colab-та ашып орындауға болады. Алғашқы ұяшық PySpark орнатады.

## Неге бұл Big Data жобасы?

Spark DataFrame деректерді бөліктерге бөліп өңдейді және MLlib pipeline есептеуді кластерге масштабтауға арналған. Кодтағы `--rows` параметрі арқылы жазба санын жүз мыңнан бірнеше миллионға дейін өзгертуге болады.

## Қорытынды

Бұл жобада Big Data аналитикасында машиналық оқыту әдістерін қолдану мақсатында клиенттердің қызметтен кетуін болжау есебі қарастырылды. Logistic Regression, Random Forest және Gradient Boosted Trees модельдеріне салыстырмалы талдау жүргізілді.

Сандық эксперимент нәтижесінде **Random Forest моделі 73,17% Accuracy көрсеткішімен** ең жоғары жіктеу дәлдігіне қол жеткізді. Бұл Gradient Boosted Trees моделінен 1,12 пайыздық пунктке, ал Logistic Regression моделінен 2,69 пайыздық пунктке жоғары.

Ал **Gradient Boosted Trees моделі ROC-AUC бойынша 78,23% және F1-score бойынша 51,23%** нәтижесін көрсетіп, осы екі метрика бойынша үздік болды. Оның ROC-AUC көрсеткіші Random Forest моделінен 0,36 пайыздық пунктке жоғары.

Осылайша, жүргізілген тәжірибеде жалпы жіктеу дәлдігі бойынша Random Forest, ал кластарды ажырату қабілеті мен F1-score бойынша Gradient Boosted Trees тиімдірек екені анықталды. Apache Spark MLlib технологиясы аталған алгоритмдерді үлкен көлемді деректерге масштабтауға мүмкіндік береді.
