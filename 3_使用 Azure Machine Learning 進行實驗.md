1. AutoML 會自動對數值資料套用規模調整和正規化，以協助防止任何大規模的特徵支配定型
2. 想要使用整合的特徵化函式，可加以自訂。 例如，您可以指定針對特定功能所應使用的插補法
3. AutoML 實驗完成之後，您將能夠檢閱套用了哪些規模調整和正規化方法。 如果 AutoML 偵測到資料發生任何問題 (例如遺漏值或類別不平衡)，您也會收到通知
4. 您可以選擇禁止選取個別演算法；如果您知道資料不適合特定類型的演算法，這很有用
5. primary_metric: 決定最佳模型的目標效能標準
6. 有數個選項可設定 AutoML 實驗的限制：
    
    ```markdown
    classification_job.set_limits(
    timeout_minutes=60,
    trial_timeout_minutes=20,
    max_trials=5,
    enable_early_termination=True,
    )
    ```
    
    - timeout_minutes：完整 AutoML 實驗終止後的分鐘數
    - trial_timeout_minutes：一項試驗可能需要的分鐘數上限
    - max_trials：試驗數目上限，或將定型的模型數目上限
    - enable_early_termination：如果分數未在短期內改善，是否結束實驗
7. 分類模型支援的三個資料護欄
    - 類別平衡偵測
    - 遺漏特徵值插補
    - 高基數特徵偵測
8. MLflow
    - 一個開放原始碼程式庫，可用來追蹤和管理機器學習實驗
    - 可記錄參數、計量和成品
