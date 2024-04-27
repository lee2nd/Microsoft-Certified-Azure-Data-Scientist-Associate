1. 指令碼很適合用於在實際執行環境中進行測試和自動化。 若要建立生產就緒指令碼，您必須
    - 清除非必要的程式碼
    - 將程式碼重構為函數
    - 在終端中測試指令碼(python [train.py](http://train.py/))
2. 用 MLflow 追蹤機器學習作業有兩個選項：
    - 使用 mlflow.autolog() 啟用自動記錄
    - 使用 mlflow.log_* 透過記錄函式追蹤自訂計量
        
        ```markdown
        import mlflow
        
        reg_rate = 0.1
        mlflow.log_param("Regularization rate", reg_rate)
        ```
        
        - mlflow.log_param()：記錄單一索引鍵/值參數。 針對您想要記錄的輸入參數使用此函式
        - mlflow.log_metric()：記錄單一索引鍵/值計量。 值必須是數字。 針對您想要與執行一起儲存的任何輸出使用此函式。
        - mlflow.log_artifact()：記錄檔案。 針對您想要記錄的任何繪圖使用此函式，請先儲存為影像檔。
3. yaml
    
    ```markdown
    name: mlflow-env
    channels:
      - conda-forge
    dependencies:
      - python=3.8
      - pip
      - pip:
        - numpy
        - pandas
        - scikit-learn
        - matplotlib
        - mlflow
        - azureml-mlflow
    ```
    
4. 您必須在 Azure Machine Learning 中將定型指令碼提交為作業
5. 多次工作執行可以分組到一個實驗中
6. 可以視需要在多個實驗之間執行搜尋。 如果您想要在不同實驗中 (由不同人員或專案反覆項目) 比較相同模型的執行狀況
    - mlflow.search_runs(exp.experiment_id)
    - mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)
    - mlflow.search_runs(exp.experiment_id, filter_string="params.num_boost_round='100'", max_results=2)
7. 定義搜尋空間
    - 離散超參數
        - QUniform(min_value, max_value, q)
        - QLogUniform(min_value, max_value, q)
        - QNormal(mu, sigma, q)
        - QLogNormal(mu, sigma, q)
    - 連續超參數
        - Uniform(min_value, max_value)
        - LogUniform(min_value, max_value)
        - Normal(mu, sigma)
        - LogNormal(mu, sigma)
    - EX:
    batch_size 超參數可以有 16、32 或 64 的值，且 learning_rate 超參數可以有常態分佈的任何值，其平均數為 10，標準差為 3
        
        ```markdown
        from azure.ai.ml.sweep import Choice, Normal
        
        command_job_for_sweep = job(
        batch_size=Choice(values=[16, 32, 64]),
        learning_rate=Normal(mu=10, sigma=3),
        )
        ```
        
8. 取樣
    - Grid search
        - 所有超參數都是離散
    - Random search
        - 可能會是離散和連續的值的混合
        - Sobol 是一種隨機取樣類型，可讓您使用種子。 當您新增種子時，可以重現整理作業，而且搜尋空間分佈會更平均地散佈
            
            ```markdown
            from azure.ai.ml.sweep import RandomParameterSampling
            
            sweep_job = command_job_for_sweep.sweep(
            sampling_algorithm = RandomParameterSampling(seed=123, rule="sobol"),
            ...
            )
            ```
            
    - 貝氏
9. Early stop
    - evaluation_interval
    - delay_evaluation
    - 新的模型可能只會比先前的模型稍微好一些。 若要判斷模型應該執行得比先前試用更好的程度，有三個選項可供提早終止
        - Bandit 原則
        - 中位數停止原則
        - 截斷選取原則
10. 超參數的 search 可以寫在命令列中
    
    ```markdown
    from azure.ai.ml import command
    
    # configure command job as base
    
    job = command(
        code="./src",
        command="python train.py --regularization ${{inputs.reg_rate}}",
        inputs={"reg_rate": 0.01},
        environment="AzureML-sklearn-0.24-ubuntu18.04-py37-cpu@latest",
        compute="aml-cluster",
    )
    ```
    
    ```markdown
    # 法一: 手動寫
    from azure.ai.ml.sweep import Choice
    command_job_for_sweep = job(
        reg_rate=Choice(values=[0.01, 0.1, 1]),
    )
    ```
    
    ```markdown
    # 法二: 自動化 search
    from azure.ai.ml import MLClient
    
    # apply the sweep parameter to obtain the sweep_job
    sweep_job = command_job_for_sweep.sweep(
        compute="aml-cluster",
        sampling_algorithm="grid",
        primary_metric="Accuracy",
        goal="Maximize",
    )
    
    # set the name of the sweep job experiment
    sweep_job.experiment_name="sweep-example"
    
    # define the limits for this sweep
    sweep_job.set_limits(max_total_trials=4, max_concurrent_trials=2, timeout=7200)
    
    # submit the sweep
    returned_sweep_job = ml_client.create_or_update(sweep_job)
    ```
    
11. 元件(components)可讓您建立可重複使用的指令碼
    - 為了建置管線(管線是機器學習工作的工作流程，其中每個工作都會定義為元件)
    - 為了共用現成可用的程式碼
    - 可以建立元件來將程式碼 (以您慣用的語言) 儲存在工作區內
    - 元件可以包含 Python 指令碼，以將資料正規化、將機器學習模型定型，或評估模型
    - 建立元件需要
        - 您要執行的工作流程的指令碼
        - 定義元件的中繼資料、介面，和命令、程式碼和環境的 YAML 檔案
    - 可以循序或以平行方式排列，讓您能夠建置複雜的流程邏輯來協調機器學習作業
    
    ```markdown
    $schema: https://azuremlschemas.azureedge.net/latest/commandComponent.schema.json
    name: prep_data
    display_name: Prepare training data
    version: 1
    type: command
    inputs:
      input_data: 
        type: uri_file
    outputs:
      output_data:
        type: uri_file
    code: ./src
    environment: azureml:AzureML-sklearn-0.24-ubuntu18.04-py37-cpu@latest
    command: >-
      python prep.py 
      --input_data ${{inputs.input_data}}
      --output_data ${{outputs.output_data}}
    ```
    
12. 問題排查
    - 如果管線本身的設定發生問題，您可以在管線作業的輸出和記錄中找到詳細資訊
    - 如果元件設定有問題，您可以在失敗元件的子作業輸出和記錄中找到詳細資訊
13. 管線特別適用于自動化機器學習模型的 re-train
    
    ```markdown
    from azure.ai.ml.entities import RecurrenceTrigger
        
    schedule_name = "run_every_minute"
    
    recurrence_trigger = RecurrenceTrigger(
    frequency="minute",
    interval=1,
    )
    ```
