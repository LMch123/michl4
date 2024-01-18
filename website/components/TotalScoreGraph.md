# TotalScoreGraph

## Props

<!-- @vuese:TotalScoreGraph:props:start -->
|Name|Description|Type|Required|Default|
|---|---|---|---|---|
|chartData|Data for rendering the chart|`Object`|`false`|{"labels":[],"datasets":[]}|
|questionnaireTitle|Title of the questionnaire for determining color scales|`String`|`false`|-|

<!-- @vuese:TotalScoreGraph:props:end -->


## Computed

<!-- @vuese:TotalScoreGraph:computed:start -->
|Computed|Type|Description|From Store|
|---|---|---|---|
|barChartData|-|Prepare data for rendering the Bar chart.|No|
|barColors|-|Determine colors for each bar in the chart based on total scores.|No|
|chartOptions|-|Chart options configuration.|No|

<!-- @vuese:TotalScoreGraph:computed:end -->


