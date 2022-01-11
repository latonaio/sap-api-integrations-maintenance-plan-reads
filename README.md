# sap-api-integrations-maintenance-plan-reads 
sap-api-integrations-maintenance-plan-reads は、外部システム(特にエッジコンピューティング環境)をSAPと統合することを目的に、SAP API で保全計画データ を取得するマイクロサービスです。    
sap-api-integrations-maintenance-plan-reads には、サンプルのAPI Json フォーマットが含まれています。   
sap-api-integrations-maintenance-plan-reads は、オンプレミス版である（＝クラウド版ではない）SAPS4HANA API の利用を前提としています。クラウド版APIを利用する場合は、ご注意ください。   
https://api.sap.com/api/OP_API_MAINTENANCEPLAN_0001/overview  

## 動作環境  
sap-api-integrations-maintenance-plan-reads は、主にエッジコンピューティング環境における動作にフォーカスしています。  
使用する際は、事前に下記の通り エッジコンピューティングの動作環境（推奨/必須）を用意してください。  
・ エッジ Kubernetes （推奨）    
・ AION のリソース （推奨)    
・ OS: LinuxOS （必須）    
・ CPU: ARM/AMD/Intel（いずれか必須）    

## クラウド環境での利用
sap-api-integrations-maintenance-plan-reads は、外部システムがクラウド環境である場合にSAPと統合するときにおいても、利用可能なように設計されています。 

## 本レポジトリ が 対応する API サービス
sap-api-integrations-maintenance-plan-reads が対応する APIサービス は、次のものです。

* APIサービス概要説明 URL: https://api.sap.com/api/OP_API_MAINTENANCEPLAN_0001/overview   
* APIサービス名(=baseURL): API_MAINTENANCEPLAN  

## 本レポジトリ に 含まれる API名
sap-api-integrations-maintenance-plan-reads には、次の API をコールするためのリソースが含まれています。  

* MaintenancePlan（保全計画 - ヘッダ）※保全計画の詳細データを取得するために、ToStragetyCycle、ToMaintenanceCycle、ToCallObject、と合わせて利用されます。
* ToStragetyCycle(保全計画 - 保全方針周期 ※To)
* ToMaintenanceCycle(保全計画 - 保全周期 ※To)
* ToCallObject（保全計画 - 保全コール対象 ※To）

## API への 値入力条件 の 初期値
sap-api-integrations-maintenance-plan-reads において、API への値入力条件の初期値は、入力ファイルレイアウトの種別毎に、次の通りとなっています。  

### SDC レイアウト

* inoutSDC.MaintenancePlan.MaintenancePlan（保全計画）

## SAP API Bussiness Hub の API の選択的コール

Latona および AION の SAP 関連リソースでは、Inputs フォルダ下の sample.json の accepter に取得したいデータの種別（＝APIの種別）を入力し、指定することができます。  
なお、同 accepter にAll(もしくは空白)の値を入力することで、全データ（＝全APIの種別）をまとめて取得することができます。  

* sample.jsonの記載例(1)  

accepter において 下記の例のように、データの種別（＝APIの種別）を指定します。  
ここでは、"Header" が指定されています。    
  
```
	"api_schema": "MaintenancePlan",
	"accepter": ["Header"],
	"maintenance_plan": "1",
	"deleted": false
```
  
* 全データを取得する際のsample.jsonの記載例(2)  

全データを取得する場合、sample.json は以下のように記載します。  

```
	"api_schema": "MaintenancePlan",
	"accepter": ["All"],
	"maintenance_plan": "1",
	"deleted": false
```

## 指定されたデータ種別のコール

accepter における データ種別 の指定に基づいて SAP_API_Caller 内の caller.go で API がコールされます。  
caller.go の func() 毎 の 以下の箇所が、指定された API をコールするソースコードです。  

```
func (c *SAPAPICaller) AsyncGetMaintenancePlan(maintenancePlan string, accepter []string) {
	wg := &sync.WaitGroup{}
	wg.Add(len(accepter))
	for _, fn := range accepter {
		switch fn {
		case "Header":
			func() {
				c.Header(maintenancePlan)
				wg.Done()
			}()
		default:
			wg.Done()
		}
	}

	wg.Wait()
}
```
## Output  
本マイクロサービスでは、[golang-logging-library](https://github.com/latonaio/golang-logging-library) により、以下のようなデータがJSON形式で出力されます。  
以下の sample.json の例は、SAP 保全計画　の　ヘッダデータ が取得された結果の JSON の例です。  
以下の項目のうち、"MaintenancePlan" ～ "to_Item" は、/SAP_API_Output_Formatter/type.go 内 の Type Header {} による出力結果です。"cursor" ～ "time"は、golang-logging-library による 定型フォーマットの出力結果です。  

```
{
	"cursor": "/Users/latona2/bitbucket/sap-api-integrations-maintenance-plan-reads/SAP_API_Caller/caller.go#L53",
	"function": "sap-api-integrations-maintenance-plan-reads/SAP_API_Caller.(*SAPAPICaller).Header",
	"level": "INFO",
	"message": [
		{
			"MaintenancePlan": "1",
			"MaintenancePlanDesc": "Mechanical Inspection for Pump",
			"CreationDate": "/Date(1498435200000)/",
			"LastChangeDate": "/Date(1498435200000)/",
			"MaintenanceStrategy": "EM_01",
			"SchedulingDuration": "360",
			"SchedulingDurationUnit": "DAY",
			"NumberOfMaintenanceItems": "1",
			"CycleModificationRatio": "1.00",
			"MaintPlanSchedgIndicator": "",
			"MaintenancePlanInternalID": "WO000000000001",
			"MaintenanceCall": 12,
			"MaintenancePlanCategory": "PM",
			"MaintPlanFreeDefinedAttrib": "",
			"BasicStartDate": "/Date(1498435200000)/",
			"SchedulingStartDate": "",
			"SchedulingStartTime": "PT00H00M00S",
			"MaintPlanStartCntrReadingValue": "",
			"MaintPlnStrtBufDurationInDays": "0",
			"MaintPlanStartBufferUnit": "DAY",
			"FactoryCalendar": "",
			"LateCompletionShiftInPercent": "100",
			"LateCompletionTolerancePercent": "10",
			"EarlyCompletionShiftInPercent": "100",
			"EarlyCompletionTolerancePct": "10",
			"MaintPlanLogicalOperatorCode": "",
			"SchedulingEndDate": "",
			"MaintPlanEndCntrReadingValue": "",
			"LastChangeDateTime": "",
			"MultipleCounterPlanShiftFactor": "",
			"MaintenanceLeadFloatInDays": "0",
			"MaintenancePlanCallObject": "",
			"MaintenancePlanSystemStatus": "I0001",
			"to_StrategyCycle": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_MAINTENANCEPLAN/MaintenancePlan('1')/to_StrategyCycle",
			"to_MaintenanceCycle": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_MAINTENANCEPLAN/MaintenancePlan('1')/to_MaintenanceCycle",
			"to_Item": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_MAINTENANCEPLAN/MaintenancePlan('1')/to_Item"
		}
	],
	"time": "2021-12-28T18:30:38.683686+09:00"
}

```
