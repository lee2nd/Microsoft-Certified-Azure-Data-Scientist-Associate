1. train 完後，超參已經找出最佳解，model 也存好，準備要部署機器學習模型，以便將模型與應用程式整合，可向 MLflow 註冊模型，輕鬆地將模型部署到批次或線上端點
2. Azure Machine Learning 和 MLflow 目前不支援某些類型的模型。 在此情況下，您可以註冊 custom 模型
3. 為什麼要使用 MLflow?
您可以使用 MLflow 來註冊模型。 MLflow 會將模型封裝標準化，這表示 MLflow 模型可以輕鬆地跨不同的工作流程匯入或匯出，如果您想要將模型匯出至用於生產環境的另一個工作區，您可以使用 MLflow 模型輕鬆地執行這項操作
4. MLmodel 檔案是有關於如何載入及使用模型的單一事實來源
    
    ```markdown
    artifact_path: classifier
    flavors:
      fastai:
        data: model.fastai
        fastai_version: 2.4.1
      python_function:
        data: model.fastai
        env: conda.yaml
        loader_module: mlflow.fastai
        python_version: 3.8.12
    model_uuid: e694c68eba484299976b06ab9058f636
    run_id: e13da8ac-b1e6-45d4-a9b2-6a0a5cfac537
    signature:
      inputs: '[{"type": "tensor",
                 "tensor-spec": 
                     {"dtype": "uint8", "shape": [-1, 300, 300, 3]}
               }]'
      outputs: '[{"type": "tensor", 
                  "tensor-spec": 
                     {"dtype": "float32", "shape": [-1,2]}
                }]'
    ```
    
5. 簽章有兩種類型：
    - 資料行型：用於具有 pandas.Dataframe 作為輸入的表格式資料
    - Tensor 型：用於 n 維陣列或張量 (通常用於非結構化資料，例如文字或影像)，且以 numpy.ndarray 作為輸入
6. 部署模型時，簽章的輸入和輸出很重要。 當您針對 MLflow 模型使用 Azure Machine Learning 的無程式碼部署時，將會強制執行簽章中所設定的輸入和輸出。 換句話說，當您將資料傳送至已部署的 MLflow 模型時，預期的輸入和輸出必須符合簽章中所定義的結構描述
7. Python 函式變體(可以想像成 input、output 的 table schema)
    
    ```markdown
    artifact_path: pipeline
    flavors:
      python_function:
        env:
          conda: conda.yaml
          virtualenv: python_env.yaml
        loader_module: mlflow.sklearn
        model_path: model.pkl
        predict_fn: predict
        python_version: 3.8.5
      sklearn:
        code: null
        pickled_model: model.pkl
        serialization_format: cloudpickle
        sklearn_version: 1.2.0
    mlflow_version: 2.1.0
    model_uuid: b8f9fe56972e48f2b8c958a3afb9c85d
    run_id: 596d2e7a-c7ed-4596-a4d2-a30755c0bfa5
    signature:
      inputs: '[{"name": "age", "type": "long"}, {"name": "sex", "type": "long"}, {"name":
        "cp", "type": "long"}, {"name": "trestbps", "type": "long"}, {"name": "chol",
        "type": "long"}, {"name": "fbs", "type": "long"}, {"name": "restecg", "type":
        "long"}, {"name": "thalach", "type": "long"}, {"name": "exang", "type": "long"},
        {"name": "oldpeak", "type": "double"}, {"name": "slope", "type": "long"}, {"name":
        "ca", "type": "long"}, {"name": "thal", "type": "string"}]'
      outputs: '[{"name": "target", "type": "long"}]'
    ```
    
8. 已註冊的模型可以用名稱和版本來識別。 每次您使用與既有名稱相同的名稱註冊模型時，登錄會遞增版本。 您也可以新增更多中繼資料標籤，以更輕鬆地搜尋特定模型
9. 您可以註冊的模型有三種類型
    - MLflow：使用 MLflow 定型和追蹤的模型。 建議用於標準使用案例
    - 自訂：Azure Machine Learning 目前不支援的具有自訂標準的模型類型
    - Triton：深度學習工作負載的模型類型。 通常用於 TensorFlow 和 PyTorch 模型部署
10. 定型模型
    
    ```markdown
    from azure.ai.ml.entities import Model
    from azure.ai.ml.constants import AssetTypes
    
    job_name = returned_job.name
    
    run_model = Model(
        path=f"azureml://jobs/{job_name}/outputs/artifacts/paths/model/",
        name="mlflow-diabetes",
        description="Model created from run.",
        type=AssetTypes.MLFLOW_MODEL,
    )
    # Uncomment after adding required details above
    ml_client.models.create_or_update(run_model)
    ```
11. 負責任 AI 儀表板中找到下列深入解析：
    - 錯誤分析
        - 錯誤樹狀圖：可讓您探索哪些子群組組合會導致模型產生更多誤判預測
        - 錯誤熱度圖：在一或兩個功能的規模上呈現模型錯誤的格線概觀
    - 解釋
    - 反事實
    - 原因分析
