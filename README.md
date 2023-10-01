# sts_memo
memo
クラス図の説明
　　継承(インターフェースの実装)
　　集約
　　多重度

クラス
　　継承・インターフェース
　　クラス・インスタンス


本題
　　ビューとテストの分離(コントローラとモニタを介してアクセスする)
　　　　コントローラとモニタはAutoTestManagerから取得する
　　　　実行操作はコントローラで指示する
　　AutoTest全体の構造
　　　　AutoTestManagerが自動試験の大本クラス(インスタンスは一つだけ)
　　　　TestExecuterは指定されたシナリオの数(リスト内のシナリオ数)分作成される
　　　　TestExecuter一つが一つのシナリオファイルで実行されるテスト全体をモデル化したもの
　　　　TestExecuterはScenarioと複数のITestExecution(模擬および外部インターフェース)の参照を持つ
　　　　Scenarioは複数のIScenarioAnalyzer(模擬および外部インターフェース)を持つ

　　　　AutoTestManager.Init()
　　　　　　各模擬、外部インターフェースのインスタンス生成
　　　　　　各模擬の定義ファイルをロード(LoadDefinition())
　　　　　　各模擬に必要な外部インターフェースをセット(SetXXXInterface)

　　　　AutoTestManager.DoAutoTest()
　　　　　　TestExecuterをシナリオ数分作成し、各テストを順番に実行する。

　　　　Scenario.Load()
　　　　　　1.引数で指定されたパスのシナリオファイルを読み込み、以下を読み込む
　　　　　　　　(1) [Initialization]セクションのファイルパス→InitializationInfo
　　　　　　　　(2) [Scenario]セクションの実行要素群→ScenarioElementのリスト
　　　　　　2.ScenarioElementを担当する模擬・外部インターフェースに振り分ける。
　　　　　　　　(1) 模擬ごとにリストを作成
　　　　　　　　(2) IScenarioAnalyzer.GetHandingDeviceCategory()で得られたリストに従って判定する
　　　　　　3.模擬・外部インターフェースの解析を実行する
　　　　　　　　(1) IScenarioAnalyzer.AnalyzeScenario()を実行
　　　　
　　　　TestExecuter.DoTest()
　　　　　　1シナリオ分のテストを実行する。
　　　　　　1.シナリオ解析
　　　　　　　　(1) Scenario.Load()を実行
　　　　　　2.テスト実行初期処理
　　　　　　　　(1) 模擬・外部インターフェースの初期化
　　　　　　　　(2) シナリオ時刻の初期化
　　　　　　　　(3) OFPリブート
　　　　　　3.テスト実行(シナリオ終了時刻まで繰り返し)
　　　　　　　　(1) シナリオ時刻更新
　　　　　　　　(2) 各模擬・外部インターフェースの周期処理(ITestExecute.DoTimeTickProc())実行
　　　　　　　　(3) ITestExecute.InContinuable()で継続可否を判定(継続不可なら終了)
　　　　　　　　(4) (1)に戻る

　　各模擬と外部インターフェースの実装
　　　　主な実装すべき関数
　　　　IScenarioAalyzer.ScenarioAnalyze()
　　　　　　引数：初期データパス(シナリオの[Initialization]セクションで指定されたパス群)
　　　　　　引数：各模擬用に選別されたシナリオ実行用
　　　　　　実装すべき内容
　　　　　　　　シナリオ解析
　　　　　　　　基本構造は以下
　　　　　　　　　　(1) 渡されたScenarioElementのリストから、テスト実行用内部データを作成する
　　　　　　　　　　
　　　　ITestExecution.DoTimeTickProc()
　　　　　　引数なし
　　　　　　実装すべき内容
　　　　　　　　いわゆる周期実行関数
　　　　　　　　基本構造は以下
　　　　　　　　　　(1) 現在のシナリオ時刻を確認
　　　　　　　　　　(2) テスト実行用内部データから、実行すべき処理を実行(set、transmit、即時verify)
　　　　　　　　　　(3) エラーがあれば継続可否に不可を設定
　　　　
　　　　ITestExecution.IsContinuable()
　　　　　　引数なし
　　　　　　実装すべき内容
　　　　　　　　Testの続行可否を返す。
　　　　　　　　(即時verifyのエラーまたは外部インターフェースのエラーがあった場合、続行不可)

　　　　LoadDefinition()
　　　　　　引数なし
　　　　　　実装すべき内容
　　　　　　　　各模擬で必要な定義ファイルをロードする。
　　　　　　　　定義ファイルのパスはSystemCommonSettingから取得する。
　　　　　　　　
　　　　SetXXXInterface()
　　　　　　引数：模擬に必要な外部インターフェースの参照
　　　　　　実装すべき内容
　　　　　　　　外部インターフェースの参照を保持する。
