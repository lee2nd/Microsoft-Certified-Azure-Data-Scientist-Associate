1. 若要取用模型，您必須部署模型。 部署模型的方式之一，是將其與服務整合，讓應用程式能夠針對個別或一小組資料點要求立即 (或即時) 的預測
2. 端點是一個 HTTPS 端點，您可以將資料傳送到其中，該端點會以 (將近) 立即的速度傳回回應
3. Azure Machine Learning 中有兩種線上端點
    - 受控線上端點：Azure Machine Learning 會管理所有基礎結構
    - Kubernetes 線上端點：使用者管理的是提供必要基礎結構的 Kubernetes 叢集
4. 將 MLFlow 模型部署到線上端點時，不需要提供評分指令碼和環境，因為這兩者會自動產生
5. 要將模型部署至受控線上端點，您需要指定四件事
    - 模型資產，例如模型 pickle 檔案，或 Azure Machine Learning 中的已註冊模型
    - 評分指令碼，這會載入模型
    - 環境，這會列出必須安裝在端點計算上的所有必要套件
    - 計算組態包括所需的計算大小和調整設定，以確保您可以處理端點將接收的要求數
6. 單一端點可以有多個部署
7. 藍/綠部署
    - 實驗之後，您可以選取效能最佳的模型。 您會針對第一版的模型使用藍色部署。 收集新資料時，可能會重新定型模型，並將新版本註冊到 Azure Machine Learning 工作區。 若要測試新模型，您可以針對第二版模型使用綠色部署
    - 您可以決定要將多少流量轉送至每個已部署的模型。 如此一來，您可以切換到新版本的模型，而不中斷提供給取用者的服務
8. 建立端點
    
    ```markdown
    from azure.ai.ml.entities import ManagedOnlineEndpoint
    
    # create an online endpoint
    endpoint = ManagedOnlineEndpoint(
        name="endpoint-example",
        description="Online endpoint",
        auth_mode="key",
    )
    
    ml_client.begin_create_or_update(endpoint).result()
    ```
    
9. 部署 (並自動註冊) 模型
    
    ```markdown
    from azure.ai.ml.entities import Model, ManagedOnlineDeployment
    from azure.ai.ml.constants import AssetTypes
    
    # create a blue deployment
    model = Model(
        path="./model",
        type=AssetTypes.MLFLOW_MODEL,
        description="my sample mlflow model",
    )
    
    blue_deployment = ManagedOnlineDeployment(
        name="blue",
        endpoint_name="endpoint-example",
        model=model,
        instance_type="Standard_F4s_v2",
        instance_count=1,
    )
    
    ml_client.online_deployments.begin_create_or_update(blue_deployment).result()
    ```
    
    - instance_type：要使用的虛擬機器 (VM) 大小
    - instance_count：要使用的執行個體數目
10. 測試部屬
    
    ```markdown
    # sample data
    {
      "data":[
          [0.1,2.3,4.1,2.0], // 1st case
          [0.2,1.8,3.9,2.1],  // 2nd case,
          ...
      ]
    }
    
    # test the blue deployment with some sample data
    response = ml_client.online_endpoints.invoke(
        endpoint_name=online_endpoint_name,
        deployment_name="blue",
        request_file="sample-data.json",
    )
    
    if response[1]=='1':
        print("Yes")
    else:
        print ("No")
    ```
    
11. 批次推斷是用來以非同步方式將預測模型套用至多個案例 (可以將多個模型部署到批次端點)，並將結果寫入檔案或資料庫
12. 可以從另一個服務觸發批次評分作業，例如 Azure Synapse Analytics 或 Azure Databricks
13. 建立批次端點
    
    ```markdown
    # create a batch endpoint
    endpoint = BatchEndpoint(
        name="endpoint-example",
        description="A batch endpoint",
    )
    
    ml_client.batch_endpoints.begin_create_or_update(endpoint)
    ```
    
14. 次評分作業以平行批次方式處理新資料，您需要佈建具有**多個最大執行個體(max_instances=4)的計算叢集**
    
    ```markdown
    from azure.ai.ml.entities import AmlCompute
    
    cpu_cluster = AmlCompute(
        name="aml-cluster",
        type="amlcompute",
        size="STANDARD_DS11_V2",
        min_instances=0,
        max_instances=4,
        idle_time_before_scale_down=120,
        tier="Dedicated",
    )
    
    cpu_cluster = ml_client.compute.begin_create_or_update(cpu_cluster)
    ```
    
15. 將 MLflow 模型部署至批次端點
    
    ```markdown
    from azure.ai.ml.entities import BatchDeployment, BatchRetrySettings
    from azure.ai.ml.constants import BatchDeploymentOutputAction
    
    deployment = BatchDeployment(
        name="forecast-mlflow",
        description="A sales forecaster",
        endpoint_name=endpoint.name,
        model=model,
        compute="aml-cluster",
        instance_count=2,
        max_concurrency_per_instance=2,
        mini_batch_size=2,
        output_action=BatchDeploymentOutputAction.APPEND_ROW,
        output_file_name="predictions.csv",
        retry_settings=BatchRetrySettings(max_retries=3, timeout=300),
        logging_level="info",
    )
    ml_client.batch_deployments.begin_create_or_update(deployment)
    ```
    
    - `instance_count`：用於產生預測的計算節點計數
    - `max_concurrency_per_instance`：每個計算節點執行的平行評分指令碼數目上限
    - `mini_batch_size`：每個評分指令碼執行所傳遞的檔案數目
    - `output_action`：預測的處理方式：`summary_only` 或 `append_row`
    - `output_file_name`：如果您為 `output_action` 選擇 `append_row`，則為預測附加至其中的檔案
16. 批次評分作業會以管線作業的形式執行。 如果您想要針對管線作業進行疑難排解，可以檢閱其詳細資料和管線作業本身的輸出和記錄
17. outputs+logs 可以看錯誤訊息，logs/user/ 資料夾會包含三個檔案
    - `job_error.txt`：摘要說明指令碼中的錯誤
    - `job_progress_overview.txt`：提供目前為止處理之迷你批次數目的高階資訊
    - `job_result.txt`：顯示在評分指令碼中呼叫 `init()` 和 `run()` 函式時發生的錯誤
