1. Azure Machine Learning 工作區
會儲存所有訓練作業的歷程記錄，包括記錄、計量、輸出，以及程式碼的快照集
2. Azure 儲存體帳戶：儲存工作區中使用的檔案和筆記本，以及儲存作業和模型的中繼資料
3. Azure Key Vault：用來安全管理工作區所使用的祕密，例如驗證金鑰和認證
4. Application Insights：用來監視工作區中的預測服務
5. Azure Container Registry：視需要建立，以儲存 Azure Machine Learning 環境的映像
6. 角色型存取控制 (RBAC)
7. Compute 是最重要的資源，但也可能是最耗費成本的資源。 因此，最佳做法是只允許系統管理員建立和管理計算資源。 不得允許資料科學家編輯計算
8. Azure Machine Learning 工作區中有五種類型的 Compute
    - 計算執行個體：類似於雲端中的虛擬機器，並依工作區進行管理。 適合作為開發環境來執行 (Jupyter) 筆記本。
    - 計算叢集：雲端中 CPU 或 GPU 計算節點的隨選叢集 (依工作區進行管理)。 適合用於生產工作負載，因為其會自動調整以符合您的需求。
    - Kubernetes 叢集:可讓您建立或連結 Azure Kubernetes Service (AKS) 叢集。 適合於在生產案例中部署已定型的機器學習模型。
    - 附加的計算：可讓您將其他 Azure 計算資源連結至工作區，例如 Azure Databricks 或 Synapse Spark 集區。
    - 無伺服器計算：您可以用於定型作業的完全受控隨選計算
9. 當您在工作區中建立模型時，將會指定名稱和版本。 這在您部署已註冊的模型時會特別有用，版本控制可讓您追蹤想要使用的特定模型
    - pickle檔
    - 可用 Scikit-learn 或 PyTorch
10. 可以在建立管線時使用元件。 因此，元件通常代表管線中的步驟，例如將資料正規化、訓練迴歸模型，或在驗證資料集上測試已定型的模型
11. 自動化機器學習，用來探索演算法和超參數值，可以省下測試演算法和超參數的時間，會逐一查看與特徵選取配對的演算法，以尋找資料的最佳執行模型
12. Workspace 非常適合用於快速實驗，或當您想要探索過去的作業
13. Workspace 使用時機
    - 想要確認管線是否成功執行
    - 當管線作業失敗時，可以使用工作室瀏覽至記錄，並檢閱錯誤訊息
14. Azure CLI 或 Python SDK 使用時機
    - 重複率更高的工作
    - 想要自動化的工作
15. 設計工具: 拖拉式的 uml 連結(把整個ds project做視覺化呈現，no-code形式)
16. Python SDK 線到工作區。 藉由連線，您會驗證環境以與工作區互動，以建立和管理資產和資源
17. Azure CLI 好處
    - 讓建立及設定資產和資源自動化，使其可重複
    - 針對必須在多個環境 (例如開發、測試和生產) 中複寫的資產和資源，確保其一致性
    - 將機器學習資產設定併入開發人員作業 (DevOps) 工作流程，例如持續整合和持續部署 (CI/CD) 管線
18. 若要使用 Azure CLI 與 Azure Machine Learning 工作區互動，將使用 command。 每個 command 前面都會加上 az ml
    
    ```
    az ml compute create -name aml-cluster -size STANDARD_DS3_v2 -min-instances 0 -max-instances 5 -type AmlCompute -resource-group my-resource-group -workspace-name my-workspace
    ```
    
19. Azure CLI 很常搭配 YAML 使用
    
    ```markdown
    az ml compute create -file compute.yml -resource-group my-resource-group -workspace-name my-workspace
    ```
    
    compute.yml 內容如下
    
    ```markdown
    $schema: https://azuremlschemas.azureedge.net/latest/amlCompute.schema.json
    name: aml-cluster
    type: amlcompute
    size: STANDARD_DS3_v2
    min_instances: 0
    max_instances: 5
    ```
    
20. 在 Azure Machine Learning 內容中使用資料時，有三種常見的通訊協定：
    - http(s)：用於公開或私人儲存在 Azure Blob 儲存體內的資料，或公開可用的 HTTP 位置
    - abfs(s)：用於 Azure Data Lake Storage Gen 2 中的資料存放區
    - azureml：用於儲存在資料存放區中的資料
21. Azure 上建立具有現有儲存體帳戶的資料存放區時，您可以選擇兩種不同的驗證方法
    - 認證型：使用服務主體、共用存取簽章 (SAS) 權杖或帳戶金鑰來驗證儲存體帳戶的存取權
    - 以身分識別為依據：使用您的「Microsoft Entra 身分識別」或「受控識別」
22. 每個工作區都有四個內建資料存放區（兩個連線到Azure 儲存體 Blob 容器，兩個連線到Azure 儲存體檔案共用）
23. 資料資產: 不想擔心如何取得存取權，簡化對您想要使用資料的存取
24. 建立資料資產並指向儲存在本機裝置上的檔案或資料夾時，會將檔案或資料夾的複本上傳至預設資料存放區 workspaceblobstore
25. 當您的資料架構複雜或經常變更時，建議使用 MLTable 資料資產。 您不需在使用資料的每個指令碼中變更讀取資料的方式，而只需在資料資產本身中變更
26. 計算執行個體(ComputeInstance)必須在 Azure 區域 (例如西歐內) 中具有唯一的名稱。 如果名稱已經存在，則會顯示錯誤訊息
27. 可以將計算執行個體附加至筆記本以在筆記本內執行資料格
28. 計算執行個體只能指派給「一位」使用者，因為計算執行個體無法處理平行工作負載。 當您建立新的計算執行個體時，如果您有適當的權限，您可以將它指派給其他人
29. 建立 cluster
    
    ```markdown
    from azure.ai.ml.entities import AmlCompute
    
    cluster_basic = AmlCompute(
    name="cpu-cluster",
    type="amlcompute",
    size="STANDARD_DS3_v2",
    location="westus",
    min_instances=0,
    max_instances=2,
    idle_time_before_scale_down=120,
    tier="low_priority",
    )
    
    ml_client.begin_create_or_update(cluster_basic).result()
    ```
    
    - size：指定計算叢集中每個節點的虛擬機器類型。 根據 Azure 中虛擬機器的大小。 除了大小之外，您也可以指定是否要使用 CPU 或 GPU。
    - max_instances：指定計算叢集可相應放大的節點數目上限。 計算叢集可以處理的平行工作負載數目，類似於叢集可調整的節點數目。
    - tier：指定您的虛擬機器是「低優先順序」或「專用」。 設定為低優先順序可能會降低成本，因為無法向您保證可用性
30. compute cluster
    
    ```markdown
    from azure.ai.ml import command
    
    # configure job
    job = command(
    code="./src",
    command="python [diabetes-training.py](http://diabetes-training.py/)",
    environment="AzureML-sklearn-0.24-ubuntu18.04-py37-cpu@latest",
    compute="cpu-cluster",
    display_name="train-with-cluster",
    experiment_name="diabetes-training"
    )
    
    # submit job
    returned_job = ml_client.create_or_update(job)
    aml_url = returned_job.studio_url
    print("Monitor your job at", aml_url)
    ```
    
31. 檢視 Azure Machine Learning 工作區內的所有可用環境
    
    ```markdown
    envs = ml_client.environments.list()
    for env in envs:
        print(env.name)   
    ```
    
32. 檢閱特定環境的詳細資料，您可以依其已註冊的名稱擷取環境
    
    ```markdown
    env = ml_client.environments.get(name="my-environment", version="1")
    print(env)
    ```
    
33. Docker 映像可以裝載在公用登錄中，例如 Docker Hub 或私下儲存在 Azure Container Registry 中
