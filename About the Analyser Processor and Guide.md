# Introduction to our Analytical Processor

Welcome to the repository for analytical processor, a data calculation engine. We are not intending to open source this code at this point in time because the intricacy of the application would be too much. Instead we will put out concepts, configuration, and datasets that are used to manage this application and our intention is to refine this before adding user interface management capabilities for most of the configuration.

For the future, we will allow other users to supply their own configuration which we will then use within our cryptocurrency platform. We may decide to open up this application to an API. 

This application is an integral part of our upcoming cryptocurrency platform - Crypto Statto. Our intention is to compare periods and investigate values within them to try and determine whether these values are inside or outside of acceptable boundaries. When we think about technical analysis versus using AI or other heuristics to try and identify trade signals, one acceptable form is inconsistency. For example, technical analysis is undertaken manually and provides success for many practitioners of this strategy, and this may be evidenced by some very successful traders using technical analysis. Technical Analysis is in itself quite subjective and not scaleable if done by a human. For example, there are thousands of  cryptocurrencies. At the very least some mechanisms to highlight currencies of interest should exist. Some may use watch lists, scripts for websites such as TradingView. More recently, some trading websites are offering up more advanced position management, somewhat moving towards our vision of giving the trader the ability to automate their positions and define strategies upfront. Our opinion is, we want to see the market at different levels of granularity to try and get a perspective on which currencies we should focus our attention on. We would wish to automate this and feed these into trading signals for the benefit of automating our and other's trading.

We have another outstanding platform - findigl for providing data to buyers and sellers on the property market and the Analyser Processor is very much a part of that too.

If you wish to find out more please do contact info Info Rhino, we are definitely looking to collaborate with other individuals and organisations.

These notes within this document are not in any particular order, they will change over time, but please do read this and provide feedback.

The technology runs on .Net - Microsoft C# .Net Framework, but as much of our technology stack is in dotnetcore and we wouldn't see an issue in converting this. 

### Beta Mode 

We consider this application to be in "beta" mode because we do not have the necessary UI to help manage it at the moment. We are exploring that and welcome those interested in helping to achieve this outcome.

## The principle of "Open Data"

Government has coined this phrase, some may say too soon of a landgrab. "Open data" effectively means data being publicly available which is being provided by organisations mainly public facing. This is a good thing but on many on occasion the data is in unusable formats. Our view is more to think about data in terms of not residing inside a server or embedded system; storing data on disc, retaining it inside storing. Whilst we expect the data to be a specific format, by keeping the data in more open storage it is available to other applications to make use of it. This doesn't mean we cannot secure the application. what is important is that the data is available for other applications to make use of.

The analytical processor fits this principle quite well. Aside from one of the process modes of calculation engine that does interact with a database for transforming data, the rest work entirely with data on disc. One of the reasons for this is because we could see the potential for other applications to be performed certain functionality. This means that the analyser processor application could do some of the work, or other mechanisms used to process data in a distributed fashion on different containers or virtual machines. One example is the "Build Data Array" method. It is highly probable that a document store could undertake the transformation of datasets to a "Row" collection as expected by our application.

For the want of a better phrase we see "open data" means it is easily open to other applications to work with it as required. Anybody can create a process or application to produce that data that can be used elsewhere. In our world - all data is in json format, therefore.

## About this document 

This is as much for readers as for ourselves. We are making notes on how to set up the application. We are still thinking about how to structure where and how we source and process our data, and are making notes as we go.

## IRAnalyser and IRAnalyserProcessor Applications
## Introduction
As a developer, well versed in "Business Intelligence", a common part of analysing data is to undertake set based analysis of data. Typically, we perform aggregations and calculations. A key challenge with SQL, is the need to strongly type the data, load it into a database, and then write "SQL" to return data in a format which can be used in applications or other SQL statements. Alternatively, there are languages such as MDX (Multi Dimensional Expressions) which provides a richer capability than SQL.

A key challenge with SQL is collecting values from different windows to be used within a formula. It is absolutely possible for this to be performed but each requirement needs to be hand coded. It is possible to create an engine to undertake this and generate the SQL dynamically. This was considered and indeed our analytical processor has an SQL transformation for performing additional aggregation and other transformations on one or more data sets. However, our engine is focused around getting values from different time series to then be used within formulas to be potentially combined again and interpreted by tests to determine whether values are inside or outside of a range.

The specific problem I am tasked with solving - taking flattened data series json data and permitting window analysis on it, without any need for a query language (Declarative Language) or a Procedural Language. As a bonus, the intention is to perform algebra on this data to output additional metrics.

The goal is to leave an application which, whilst technical, is configurable and obvious. It is imagined that a front-end would be used to maintain the configuration, to avoid non-technical staff maintaining configuration files.

### What is Window Analysis
This will be explained more extensively, but I got this idea from Arthur Schopenhauer's "A World as Will and Idea". In the book, he covers this idea of the phenomenon, a spinning globe, where our current position is now, ahead of us is a past, and ahead of us is the future. Humans, tend not to be able to accurately recall the past, just as much as they can't predict the future, so in Schopenhauer's view, we are best trying to enjoy our present now? Of course, this isn't extolling the virtues of hedonism, but perhaps by having a better understanding of the present, we can understand the past and future better.

Well, the interesting thing with data, is it does accurately record what happened in the past, and of course, yesterday's tomorrow's future is today's present.  The application has a concept of a Past, Present, Future, and  Window. We can guess that the Window represents the entire "Past,  Present, and Future. The Window is a "Range Position" enum type.
| Past | -1hr |
|Present|0|
|  Future|+10 mins  |
| Window| Range = -1hr to + 10 minutes|

If we went back 2 weeks, and ran analysis on this 1hr 10 minute range, we can get a set of pasts, presents, and futures.

### Calculations and ActionData

```csharp 
    /// <summary>
    /// The raw time and measure data. Typically, a date and a volume. Or a date and a price.
    /// </summary>
    public class ActionData
    {
        public DateTime ActionTime{ get; set; }

        public double Value { get; set; }

        public string RangeGroup { get; set; }
    }
```

We can recognise this as a simple time series, with a feature - RangeGroup. We could see something like;
- RangeGroup = USD/BTC
- Value = 35000
- ActionTime = 17 Sep 2011 18:11:03

IRAnalyser generates a set of action data buckets separated by configured Range Positions - Past, Present, Future, (Window is calculated).

The application has calculations inbuilt. Examples being "Correlate", "Sum", "Mean", "Standard Deviation", "Slope". Please note, that calculations implements  the "ICalculate" interface which means you can add as many calculations as you like.

``` csharp
    public interface ICalculate
    {
        IErrorDefaultRetriever errorDefaultRetriever { get;}

        CalculationErrorDefault calculationErrorDefault { get; }

        double Calculate(double[,] Values);

    }
```

### Treatments
Most data points won't make sense to be calculated in their raw form. For example, if we were trying to compare BTC to ETH, we would be better using a rate or percentage.

Treatments implements the ITreatement interface
```csharp
    public interface ICalculate
    {
        IErrorDefaultRetriever errorDefaultRetriever { get;}

        CalculationErrorDefault calculationErrorDefault { get; }

        double Calculate(double[,] Values);

    }
```

### The Two Dimensional Array, Calculations, and Treatments
Currently, all ActonData becomes a two dimensional array. Any timeseries data sees the Date portion converted to a double value, to permit it to be included in calculations if required. All calculations and treatments accept a 2 dimensional array of values.
- Treatments accepts a two dimensional array and return a two dimensional array (Transformation).
- Calculations accepts a two dimensional array and returns a single value.

### Date Calculations 
Those familiar with hierarchies in dimensional modelling will get the point, but if we simply take a date field and break it into minutes, months, weeks of a year, Years etc. Whilst this may seem nonsensical, I have a hunch that certain price action data may happen at particular intervals. Think about Market opening etc. 

### Birds eye view of the Analysis Applications - how they are intended to work

Any (DataSet) will contain at least one; Value, ActonTime, and RangeGroup. Sometimes, or frequently, one dataset may contain many Action Datas. Each one of these is called a "Container". Think of this like there being different values we may like to measure per dataset.
Each Container can contain a number of "RangeCalculations" we want to perform. 

Each RangePosition has one or more calculations which occurs over that range position's time period. These Range Positions can be fed into a formula which can be used to return a value. You can see the potential power of this application.

There are some conceptual considerations to overcome, but we can think of it like a calculation engine. Inputs in raw data files, are configured to permit multiple calculations which can be used in formulae without any programming. This data will be output into target data files, which can then be consumed by applications.

The goal is to be completely outside of any domain specific language or programming. Equally, because we use dependency injection - new calculations and treatments can be added to the application.

I will include some objects and configuration to hopefully explain how this should hang together. Apologies if this doesn't make sense. We will provide some better examples later. 

``` csharp
    public class RangeCalculation
    {

        public string CalculationKey { get; set; }
        public string Treatment { get; set; }
        public string Calculation { get; set; }
        public string RangePosition { get; set; }

    }

            public IEnumerable<RangePositionWindow> GetRangePositionWindows()
        {

            List<RangePositionWindow> windows = new List<RangePositionWindow>();

            windows.Add(new RangePositionWindow()
            {
                Start = new TimeSpan(0, 0, 0)
            ,
                End = new TimeSpan(00, 05, 30)
            ,
                RangeCalculations = GetRangeCalculations().Where(x => x.RangePosition == RangePosition.Past.ToString())
            ,
                RangeGroup = "Price"
            ,
                RangePosition = RangePosition.Past
             
            });

            windows.Add(new RangePositionWindow()
            {
                Start = new TimeSpan(0, 0, 0)
,
                End = new TimeSpan(00, 01,0)
,
                RangeCalculations = GetRangeCalculations().Where(x => x.RangePosition == RangePosition.Present.ToString())
,
                RangeGroup = "Price"
,
                RangePosition = RangePosition.Present

            });

            windows.Add(new RangePositionWindow()
            {
                Start = new TimeSpan(0, 0, 0)
,
                End = new TimeSpan(00, 20, 0)
,
                RangeCalculations = GetRangeCalculations().Where(x => x.RangePosition == RangePosition.Future.ToString())
,
                RangeGroup = "Price"
,
                RangePosition = RangePosition.Future

            });

            return windows;

        }

        public IEnumerable<RangeCalculation> GetRangeCalculations()
        {
            List<RangeCalculation> rangeCalculations = new List<RangeCalculation>();


            rangeCalculations.Add(new RangeCalculation()
            {
                Calculation = nameof(Max)
            ,
                CalculationKey = GetRangePositionCalcKey(RangePosition.Past, nameof(Max))
                ,
                RangePosition = RangePosition.Past.ToString()
                , Treatment = nameof(None)
                
            });

            rangeCalculations.Add(new RangeCalculation()
            {
                Calculation = nameof(Sum)
            ,
                CalculationKey = GetRangePositionCalcKey(RangePosition.Past, nameof(Sum))
                ,
                RangePosition = RangePosition.Past.ToString()
                ,
                Treatment = nameof(None)
            });

            rangeCalculations.Add(new RangeCalculation()
            {
                Calculation = nameof(Count)
            ,
                CalculationKey = GetRangePositionCalcKey(RangePosition.Past, nameof(Count))
                                ,
                RangePosition = RangePosition.Past.ToString()
                ,
                Treatment = nameof(None)

            });


            rangeCalculations.Add(new RangeCalculation()
            {
                Calculation = nameof(Count)
            ,
                CalculationKey = GetRangePositionCalcKey(RangePosition.Present, nameof(Count))
                ,
                RangePosition = RangePosition.Present.ToString()
                ,
                Treatment = nameof(None)
            });

            rangeCalculations.Add(new RangeCalculation()
            {
                Calculation = nameof(Mean)
            ,
                CalculationKey = GetRangePositionCalcKey(RangePosition.Present, nameof(Mean))
            });

            rangeCalculations.Add(new RangeCalculation()
            {
                Calculation = nameof(Max)
,
                CalculationKey = GetRangePositionCalcKey(RangePosition.Future, nameof(Max))

            });

            rangeCalculations.Add(new RangeCalculation()
            {
                Calculation = nameof(Sum)
            ,
                CalculationKey = GetRangePositionCalcKey(RangePosition.Future, nameof(Sum))
            });

            rangeCalculations.Add(new RangeCalculation()
            {
                Calculation = nameof(Count)
            ,
                CalculationKey = GetRangePositionCalcKey(RangePosition.Future, nameof(Count))
            });


            rangeCalculations.Add(new RangeCalculation()
            {
                Calculation = nameof(Count)
            ,
                CalculationKey = GetRangePositionCalcKey(RangePosition.Future, nameof(Count))
            });

            return rangeCalculations;

        }


        public string GetRangePositionCalcKey(RangePosition rangePosition, string CalculationType)
        {
            string key = $"{rangePosition.ToString()}_{CalculationType}";

            return key;

        }
```

## Configuration and execution arguments

We will create a help output in the console application in due course, plus document this better.

### Settings File
The settings file is simply the necessary information to load a collection of an enumerable type for the application. An example is the Error Defaults used when a calculation error is returned.
            Bind(typeof(ITypeCollectionFolderRetriever<>)).To(typeof(TypeCollectionFolderRetriever<>));

The App.Config contains a "SettingFile" setting.
```xml
<add key="SettingFile" value ="{'DotExtension':'.json' , 'SearchExpression': '*.json'}"/>
```
The application should always be called with the following two commandline parameters
"C:\SomePathOfSettingsFolder" "C:\SomePathOfDataFolder"


Extra parameters. 

The optional third parameter matches the data loader expected for the operation. Currently, we only have two for the 
DataArrayCalculationProcessor process mode, "RowsRetrieverFromData" or "RowsRetriever".

The optional fourth parameter will either be the Done Helper or this will default to the third.
"DummyDoneHelper" or "DoneHelper"

We may add more versions to move files too.


If we supply the 

Simple ArgsHelper class to show how it works. 
```csharp
    static public class ArgsHelper
    {

        public static string SettingsFolderPath(this string[] args)
        {
            return args[0];
        }

        public static string DataFolderPath(this string[] args)
        {
            return args[1];
        }

    }
```
##
> Written with [StackEdit](https://stackedit.io/).


## Command Line Arguments and Process Modes

The first commandline argument contains the path as to where the configuration exists

We are looking for a file called ProcessMode.mode. Configured as such. 

```json
        {
        "ProcessModes":["ProcessContainerQueryOutput"]
        }
```

```csharp
            Bind<IProcess>().To<InformationAuditor>();
            Bind<IProcess>().To<CalculationProcessor>();
            Bind<IProcess>().To<ProcessResultRangeTests>();
            Bind<IProcess>().To<ChartProcessor>();
            Bind<IProcess>().To<ProcessContainerQueryOutput>();
            Bind<IProcess>().To<DateOrdinalCalculationProcessor>();
            Bind<IProcess>().To<ConvertDataToDataArrayProcessor>();
            Bind<IProcess>().To<DataArrayCalculationProcessor>();
            Bind<IProcess>().To<ReportFormulaTestProcessor>();

```

To run one or more of the above process modes, we include the name of the class in the process modes list. Don't just run each one without knowing what they do.

### The ProcessModes.mode file and where to place it.

"C:\SomePathOfSettingsFolder" "C:\SomePathOfDataFolder" "RowsRetrieverFromData" or "RowsRetriever""RowsRetrieverFromData" or "RowsRetriever"

So an example could be ;

"C:\SomePathOfSettingsFolder" "C:\SomePathOfDataFolder" "RowsRetriever" "DoneHelper"

Our test case 
"C:\Somewhere\etc\AnalyserProcessor.exe" "C:\InfoRhino\Processing\CryptoStatto\Horizon\Hourly" "C:\InfoRhino\Processing\CryptoStatto\Horizon\Hourly\Data" "RowsRetriever" "DoneHelper"

"We expect the process modes file to be inside the data folder. This is because we may have multiple folders of data to process using common calculations.


### CalculationProcessor - ActionDataBucketResult

Applies calculations and produces an action bucket result file in a sub folder named by the range group configured.

#### Relevant classes.
CalculationProcessor > BuildRangeCalculation > ActionBucketResultHelper > To File ResourceType.ActionDataBucketResult


##### Expected configuration 

```csharp
    public class ActionTimeColumnContainer
    {
        public string ActionTimeColumn { get; set; }

        public IEnumerable<Container> Containers { get; set; }

        /// <summary>
        /// This is a master unique list of each container's range position windows. It is expected that they will be uniquely configured.
        /// </summary>
        public IEnumerable<RangePositionWindow> RangePositionWindows { get; set; }

    }
```
Which maps to an "ActionTimeColumnContainer" FileTypeMapConfig. For our configuration, we expect a file ending with "actionTimeContainer" matching the above structure.

This file contains enough configuration to apply the calculation against the data and produce an action data bucket result file.

```json
[{"ActionTimeColumn":"PriceCheckTMS","Containers":[{"ContainerName":"Interval","containerActionData":{"ActionTimeColumn":"PriceCheckTMS","ValueColumn":"percent_change_1h","RangeGroupColumn":"symbol"},"RangeCalculations":[{"CalculationKey":"Past_Mean_symbol_percent_change_1h","Treatment":"None","Calculation":"Mean","RangePosition":"Past"},{"CalculationKey":"Present_Mean_symbol_percent_change_1h","Treatment":"None","Calculation":"Mean","RangePosition":"Present"}],"RangePositionWindows":[{"RangePosition":0,"Start":"00:00:00","End":"00:10:00"},{"RangePosition":1,"Start":"00:00:00","End":"00:10:00"}]},{"ContainerName":"Interval","containerActionData":{"ActionTimeColumn":"PriceCheckTMS","ValueColumn":"percent_change_24h","RangeGroupColumn":"symbol"},"RangeCalculations":[{"CalculationKey":"Past_Mean_symbol_percent_change_24h","Treatment":"None","Calculation":"Mean","RangePosition":"Past"},{"CalculationKey":"Present_Mean_symbol_percent_change_24h","Treatment":"None","Calculation":"Mean","RangePosition":"Present"}],"RangePositionWindows":[{"RangePosition":0,"Start":"00:00:00","End":"00:10:00"},{"RangePosition":1,"Start":"00:00:00","End":"00:10:00"}]}],"RangePositionWindows":[{"RangePosition":0,"Start":"00:00:00","End":"00:10:00"},{"RangePosition":1,"Start":"00:00:00","End":"00:10:00"}]}]
```

###### Produces 
FileTypeMapConfig.json section

  {
    "ResourceType": "ActionDataBucketResult",
    "FileExtension": ".abr",
    "SearchPattern": "*.abr"
  }


We will refactor this to use a data array on disk rather than having to create it whilst processing. 
IBuildDataArray buildDataArray { get; set; }


##### Container configuration

```json 

[{"ContainerName":"Interval","containerActionData":

	{"ActionTimeColumn":"PriceCheckTMS","ValueColumn":"percent_change_1h","RangeGroupColumn":"symbol"},
	"RangeCalculations":[
						{"CalculationKey":"Past_Mean_symbol_percent_change_1h","Treatment":"None","Calculation":"Mean","RangePosition":"Past"},
						{"CalculationKey":"Present_Mean_symbol_percent_change_1h","Treatment":"None","Calculation":"Mean","RangePosition":"Present"}
						]
	,
	"RangePositionWindows":[{"RangePosition":0,"Start":"00:00:00","End":"00:10:00"},{"RangePosition":1,"Start":"00:00:00","End":"00:10:00"}]
}
,
{"ContainerName":"Interval","containerActionData":

	{"ActionTimeColumn":"PriceCheckTMS","ValueColumn":"percent_change_24h","RangeGroupColumn":"symbol"},
	"RangeCalculations":[
						{"CalculationKey":"Past_Mean_symbol_percent_change_24h","Treatment":"None","Calculation":"Mean","RangePosition":"Past"},
						{"CalculationKey":"Present_Mean_symbol_percent_change_24h","Treatment":"None","Calculation":"Mean","RangePosition":"Present"}
						]
	,
	"RangePositionWindows":[{"RangePosition":0,"Start":"00:00:00","End":"00:10:00"},{"RangePosition":1,"Start":"00:00:00","End":"00:10:00"}]
}]

```


//TODO: EXPLAIN

ProcessResultRangeTests > buildResultRangeTests - Finds the results of action bucket results and applies formula test results to produce passing or failing verifications. A simpler explanation. We search calculations and see whether those different values are within or outside of acceptance criteria.


### ProcessResultRangeTests - Formula calculator and test results

Takes an ActionDataBucketResult and applies formulae and tests on this.

#### A sample configured formula file

[
	{
		"ContainerName" : "Interval",
		"FormulaName" : "NettDifference",
		"Formula" : "((Past_Mean_symbol_percent_change_1h - Present_Mean_symbol_percent_change_1h ) / Past_Mean_symbol_percent_change_24h)", 
		"FormulaTests" : [{"Calculator" : "GreaterThan" , "TestValues" : [2]}]
	}
]

#### The output results file map
```json
  {
    "ResourceType": "ActionBucketFormulaTestResult",
    "FileExtension": ".calcresult",
    "SearchPattern": "*.calcresult"
  }
 ```

ActionBucketFormulaTestResult.calcresult

#### Sample output

Contains necessary input data so we can validate as required. Note, we only care about the "Passed" property.

```json
{"Container":"Interval","RangeGroup":"BCH","CalculationResultAndFormulaTests":[{"CalculatedFormula":{"NotZero":false,"Result":"NaN","CalculationStatus":1,"FormulaProduced":"(( 0  -  0  ) /  0 )","ContainerName":"Interval","FormulaName":"NettDifference","Formula":"((Past_Mean_symbol_percent_change_1h - Present_Mean_symbol_percent_change_1h ) / Past_Mean_symbol_percent_change_24h)","FormulaTests":[{"Calculator":"GreaterThan","TestValues":[2.0]}]},"formulaTestResults":[{"FormulaTest":{"Calculator":"GreaterThan","TestValues":[2.0]},"Input":"NaN","Passed":false}]}],"ActionBucketResults":[{"ContainerName":"Interval","RangeGroup":"BCH","ActionDataBucket":{"Tick":0,"Windows":[{"RangePosition":0,"StartTime":"2018-04-18T23:34:43","EndTime":"2018-04-18T23:44:43"},{"RangePosition":1,"StartTime":"2018-04-18T23:44:43.001","EndTime":"2018-04-18T23:54:43"},{"RangePosition":3,"StartTime":"2018-04-18T23:34:43","EndTime":"2018-04-18T23:54:43"}],"Start":"2018-04-18T23:34:43","End":"2018-04-18T23:54:43"},"ContainerActionDataResults":[{"CalculationStatus":1,"PointSeriesData":null,"CalculationResults":[{"RangePosition":0,"ContainerActionData":{"ActionTimeColumn":"PriceCheckTMS","ValueColumn":"percent_change_24h","RangeGroupColumn":"symbol"},"CalculationKey":"Past_Mean_symbol_percent_change_24h","CalculationStatus":1,"Result":0.0,"TreatedData":[[43208.983136574076,43208.987303240741],[15.1,14.65]],"OriginalData":[[43208.983136574076,43208.987303240741],[15.1,14.65]],"ActionData":[{"ActionTime":"2018-04-18T23:35:43","Value":15.1,"RangeGroup":"BCH"},{"ActionTime":"2018-04-18T23:41:43","Value":14.65,"RangeGroup":"BCH"}]}],"ProcessTime":"2023-02-19T17:11:26.7504849Z"},{"CalculationStatus":1,"PointSeriesData":null,"CalculationResults":[{"RangePosition":1,"ContainerActionData":{"ActionTimeColumn":"PriceCheckTMS","ValueColumn":"percent_change_24h","RangeGroupColumn":"symbol"},"CalculationKey":"Present_Mean_symbol_percent_change_24h","CalculationStatus":1,"Result":0.0,"TreatedData":[[43208.990081018521,43208.994247685187],[14.75,15.01]],"OriginalData":[[43208.990081018521,43208.994247685187],[14.75,15.01]],"ActionData":[{"ActionTime":"2018-04-18T23:45:43","Value":14.75,"RangeGroup":"BCH"},{"ActionTime":"2018-04-18T23:51:43","Value":15.01,"RangeGroup":"BCH"}]}],"ProcessTime":"2023-02-19T17:11:26.7504849Z"}],"ActionData":[{"ActionTime":"2018-04-18T23:34:43","Value":15.21,"RangeGroup":"BCH"},{"ActionTime":"2018-04-18T23:35:43","Value":15.1,"RangeGroup":"BCH"},{"ActionTime":"2018-04-18T23:41:43","Value":14.65,"RangeGroup":"BCH"},{"ActionTime":"2018-04-18T23:45:43","Value":14.75,"RangeGroup":"BCH"},{"ActionTime":"2018-04-18T23:51:43","Value":15.01,"RangeGroup":"BCH"}]},{"ContainerName":"Interval","RangeGroup":"BCH","ActionDataBucket":{"Tick":0,"Windows":[{"RangePosition":0,"StartTime":"2018-04-18T23:34:43","EndTime":"2018-04-18T23:44:43"},{"RangePosition":1,"StartTime":"2018-04-18T23:44:43.001","EndTime":"2018-04-18T23:54:43"},{"RangePosition":3,"StartTime":"2018-04-18T23:34:43","EndTime":"2018-04-18T23:54:43"}],"Start":"2018-04-18T23:34:43","End":"2018-04-18T23:54:43"},"ContainerActionDataResults":[{"CalculationStatus":1,"PointSeriesData":null,"CalculationResults":[{"RangePosition":0,"ContainerActionData":{"ActionTimeColumn":"PriceCheckTMS","ValueColumn":"percent_change_1h","RangeGroupColumn":"symbol"},"CalculationKey":"Past_Mean_symbol_percent_change_1h","CalculationStatus":1,"Result":0.0,"TreatedData":[[43208.983136574076,43208.987303240741],[-0.68,-1.09]],"OriginalData":[[43208.983136574076,43208.987303240741],[-0.68,-1.09]],"ActionData":[{"ActionTime":"2018-04-18T23:35:43","Value":-0.68,"RangeGroup":"BCH"},{"ActionTime":"2018-04-18T23:41:43","Value":-1.09,"RangeGroup":"BCH"}]}],"ProcessTime":"2023-02-19T17:11:18.4831896Z"},{"CalculationStatus":1,"PointSeriesData":null,"CalculationResults":[{"RangePosition":1,"ContainerActionData":{"ActionTimeColumn":"PriceCheckTMS","ValueColumn":"percent_change_1h","RangeGroupColumn":"symbol"},"CalculationKey":"Present_Mean_symbol_percent_change_1h","CalculationStatus":1,"Result":0.0,"TreatedData":[[43208.990081018521,43208.994247685187],[-0.97,-0.74]],"OriginalData":[[43208.990081018521,43208.994247685187],[-0.97,-0.74]],"ActionData":[{"ActionTime":"2018-04-18T23:45:43","Value":-0.97,"RangeGroup":"BCH"},{"ActionTime":"2018-04-18T23:51:43","Value":-0.74,"RangeGroup":"BCH"}]}],"ProcessTime":"2023-02-19T17:11:18.4831896Z"}],"ActionData":[{"ActionTime":"2018-04-18T23:34:43","Value":-0.59,"RangeGroup":"BCH"},{"ActionTime":"2018-04-18T23:35:43","Value":-0.68,"RangeGroup":"BCH"},{"ActionTime":"2018-04-18T23:41:43","Value":-1.09,"RangeGroup":"BCH"},{"ActionTime":"2018-04-18T23:45:43","Value":-0.97,"RangeGroup":"BCH"},{"ActionTime":"2018-04-18T23:51:43","Value":-0.74,"RangeGroup":"BCH"}]}]}
```


### ReportFormulaTestProcessor - Takes the results of ProcessResultRangeTests across the range groups to produce a single report fomula test output


We configure one or more ReportFormulaTestMappedConfig. This examines all the container folder Formula Tests and brings them into a single file telling us if our tests passed or not. The objective is to allow us to link test results back to whether they passed or not for a time slice and  to group by a date element. In theory, we could group by month  for example.

```csharp 

    public class ReportFormulaTestMappedConfig
    {

        public string ContainerName { get; set; }

        /// <summary>
        /// If empty, it will include all formulas by default.
        /// </summary>
        public IEnumerable<string> Formulas { get; set; }

        public RangePosition RangePosition { get; set; }

        /// <summary>
        /// This exists in the calc result file. Need to check.
        /// </summary>
        public string RangeGroupColumn { get; set; }

        /// <summary>
        /// For a RangeWindow, do we take the start, mid, end for example. This if for plotting the xaxis.
        /// </summary>
        public string TimeSlice { get; set; }

        /// <summary>
        /// Once we have a TimeSlice, are we planning to bucket the date into intervals such as "Week", "Day","Minute","Hour"?
        /// </summary>
        public string DateElement { get; set; }
    }

```

#### Example Report Formula Test 

```json
[{
  "ContainerName": "Interval",
  "Formulas": null,
  "RangePosition": "Window",
  "RangeGroupColumn": "symbol",
  "TimeSlice": "End",
  "DateElement": "Hour"
}]

```

The "Formulas" property acts as a filter on the formulas we are checking to see whether they passed or not.


#### The ReportFormulaTest file type map config

``` json
 {
    "ResourceType": "ReportFormulaTest",
    "FileExtension": ".reportformulatest",
    "SearchPattern": "*.reportformulatest"
  }
```

#### Sample data output 

Whilst the outputs are false below, you can see how we examine the tests to summarise the range group's formulae for a time period (cryptocurrency) in this instance passed or failed. This would allow us to aggregate this and determine how many times this passed.

```json

[{"ValidResult":false,"Result":"NaN","RangeGroup":"WAVES","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":23,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-18T23:54:43"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAVES","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":1,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T01:14:43.004"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAVES","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":0,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T00:34:43.002"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAVES","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":0,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T00:34:43.002"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAVES","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":0,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T00:54:43.003"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAVES","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":1,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T01:14:43.004"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAVES","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":0,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T00:14:43.001"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAVES","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":0,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T00:14:43.001"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAVES","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":23,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-18T23:54:43"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAVES","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":0,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T00:54:43.003"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAN","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":0,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T00:34:43.002"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAN","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":1,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T01:14:43.004"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAN","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":1,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T01:14:43.004"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAN","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":0,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T00:54:43.003"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAN","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":0,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T00:34:43.002"},{"ValidResult":false,"Result":"NaN","RangeGroup":"WAN","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":0,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T00:14:43.001"}]

```



8dd33713-bce5-4dd3-9277-9cb72955346c_ReportFormulaTest.reportformulatest

Interval.configFormulaTest


### ChartProcessor - Exports chart data based upon Action Bucket Data Results - the original calculated data items for a range group over a time series.

#### Produces a collection of ChartResultMapped

```csharp
    public class ChartResultMapped
    {

        public string ContainerName { get; set; }

        public IEnumerable<string> ValueColumns { get; set; }

        public IEnumerable<RangePosition> RangePositions { get; set; }

        public string RangeGroupColumn { get; set; }

        /// <summary>
        /// For a RangeWindow, do we take the start, mid, end for example. This if for plotting the xaxis.
        /// </summary>
        public string TimeSlice { get; set; }

        /// <summary>
        /// Once we have a TimeSlice, are we planning to bucket the date into intervals such as "Week", "Day","Minute","Hour"?
        /// </summary>
        public string DateElement { get; set; }


    }
```

The above is used to filter our action bucket results to allow us to chart this data.

#### ResourceType.ConfigChartResult
  {
    "ResourceType": "ConfigChartResult",
    "FileExtension": ".configChartResult",
    "SearchPattern": "*.configChartResult"
  }


### ProcessContainerQueryOutput - Database Query data outputter from source files back to disk. 
Allows us to define sql queries to operate against a SQL Server database, we are considering enhancing this. The basic mechanism for this is to allow us to use the feature rich set-based language of a RDMS without having to deploy code to either an application or by applying DDL - i.e. We don't need to have physical objects in the database. We can refererence tables on the fly to join within our query too. We set up definitions for the columns from a json dataset we wish to run our SQL against. The resultset from the query is exported back to disk which itself can be used.

Now, it will seem like quite a bit of work to set up, but we are opening up the ability to process and transform multiple result sets without having to go through extensive ETL. We can use this to make column names conform, for example - heterogenous data sources.

Our test case example uses the reportformulatest results file. We used it to take formula test results to produce chartable data for our Web Data Platform https://www.inforhino.co.uk/services/web-data-platform-launch .
Data gets uploaded to the website in chartable datasets that our web application can automatically pull into the website to display to users as charts.

SQL Server and database dependency. Nothing stops us using SQL Server Express. Or, if your web hosting has a SQL Server database, use that. We don't have to expose an Enterprise Production database server to this but all necessary security measures should be applied to reduce the risk of exploiting vulnerabilities of databases.

```json
[{"ValidResult":true,"Result":-0.025265957446808509,"RangeGroup":"IOST","RangePosition":"Window","FormulaName":"NettDifference","FormulaTests":[{"TotalTests":1,"PassedCount":0,"FailedCount":1}],"DateElement":0,"DateFormatter":"Hour","AllTestsPassed":false,"EventTime":"2018-04-19T00:34:43.002"}]
```

We define a "queryContainers.queryContainer" file which looks like this;

```json 

[
	{"ContainerName" : "Interval", "TypeName" : "ReportFormulaTest"}
]

```

"Interval" is the Container Name and subfolder of the Data Folder. "ReportFormulaTest" is a folder within the Container Data Folder. So,

```
C:\InfoRhino\Processing\CryptoStatto\Horizon\Hourly\Data\queryContainers.queryContainer
C:\InfoRhino\Processing\CryptoStatto\Horizon\Hourly\Data\Interval\ReportFormulaTest
```

Within the ReportFormulaTest folder, we set up multiple files;

- A column List file "ColumnList.txt". These columns are from the json and of course, should not be nested/contain a collection below them.
```sql
SELECT 
Ordinal 
,ValidResult 
,Result
,RangeGroup 
,RangePosition 
,FormulaName 
,DateElement 
,DateFormatter 
,AllTestsPassed  
,EventTime 
FROM 
@{DataStructureParameterName} t
```

- Set have a data script handler file "DataScript.datahandler"
We can see that we use SQL to map a json dataset to SQL.
```json
[{
  "DataTypeName": "ReportFormulaTest",
  "DataStructureParameterName": "@SourceData",
  "ParameterName": "@JSONData",
  "ResultTableName": "@ResultsData",
  "DataHandlerStatement": "SET DATEFORMAT\r 'DMY'\r DECLARE {DataStructureParameterName} AS TABLE \r        (\r        Ordinal bigint identity(1,1) PRIMARY KEY NONCLUSTERED\r          , ValidResult bit\r           ,Result numeric(30,8)\r           , RangeGroup nvarchar(512)\r           , RangePosition nvarchar(100)\r           , FormulaName nvarchar(512)\r           , DateElement int\r           , DateFormatter nvarchar(100)\r           , AllTestsPassed bit \r           , EventTime DateTime\r        ) --WITH      (MEMORY_OPTIMIZED = ON);  \r        ;\r \r \r        INSERT INTO \r        {DataStructureParameterName} \r        (\r        ValidResult \r        ,Result \r        ,RangeGroup \r        ,RangePosition \r        ,FormulaName \r        ,DateElement \r        ,DateFormatter \r        ,AllTestsPassed\r        ,EventTime\r        )\r        SELECT \r        ValidResult \r        ,Result \r        ,RangeGroup \r        ,RangePosition \r        ,FormulaName \r        ,DateElement \r        ,DateFormatter \r        ,AllTestsPassed\r        ,EventTime\r        FROM \r        OPENJSON(\r        {ParameterName}, '$'\r        )\r        WITH \r        (\r          ValidResult bit 'strict $.ValidResult'\r           ,Result numeric(30,8) 'strict $.Result'\r           , RangeGroup nvarchar(512) 'strict $.RangeGroup'\r           , RangePosition nvarchar(100) 'strict $.RangePosition'\r           , FormulaName nvarchar(512) 'strict $.FormulaName '\r           , DateElement int 'strict $.DateElement'\r           , DateFormatter nvarchar(100) 'strict $.DateFormatter'\r           , AllTestsPassed bit  'strict $.AllTestsPassed'\r           , EventTime DateTime 'strict $.EventTime'\r        )\r        t \r        \r        {FilterStatement}\r        ;\r        \r DECLARE {ResultTableName} TABLE \r (\r [JsonResult] nvarchar(max) \r ,[ReportName] nvarchar(200) \r ,[DataTypeName] nvarchar(200) \r ,[ProcessedTMS] DATETIME DEFAULT (GETDATE())\r ,[Passed] BIT \r \r );\r \r \r DECLARE @JsonResult nvarchar(max);\r        \r {OutputStatements}       \r \r SELECT * FROM {ResultTableName}",
  "FilterName": "Last7DaysTopOnes"
}
]

```

- A Queries file "Queries.qry"

```json
[
	{
	  "ReportName": "Passing Interval Test",
	  "DataTypeName": "ReportFormulaTest",
	  "Query": "SELECT \r\nSUM(CONVERT(INT,AllTestsPassed)) YAxis\r\n,RangeGroup XAxis\r\nFROM \r\n{DataStructureParameterName} t\r\nGROUP BY \r\nRangeGroup "
	}
	,
	{
	  "ReportName": "Hourly Passing Tests",
	  "DataTypeName": "ReportFormulaTest",
	  "Query": "SELECT \r\nSUM(CONVERT(INT,AllTestsPassed)) YAxis\r\n,DATEPART(HH,EventTime) XAxis\r\nFROM \r\n{DataStructureParameterName} t\r\nGROUP BY \r\nDATEPART(HH,EventTime)\r\n"
	}
	,
	{
	  "ReportName": "Daily Hourly Passing Tests",
	  "DataTypeName": "ReportFormulaTest",
	  "Query": "SELECT \r\nSUM(CONVERT(INT,AllTestsPassed)) YAxis\r\n,DATEPART(HH,EventTime) XAxis\r\n,DATEPART(dd,EventTime) [Row]\r\nFROM \r\n{DataStructureParameterName} t\r\nGROUP BY \r\nDATEPART(HH,EventTime)\r\n,DATEPART(dd,EventTime)\r\n"
	}
	,
	{
	  "ReportName": "Daily Hourly Passing Tests by Currency",
	  "DataTypeName": "ReportFormulaTest",
	  "Query": "SELECT \r\nSUM(CONVERT(INT,AllTestsPassed)) YAxis\r\n,DATEPART(HH,EventTime) XAxis\r\n,DATEPART(dd,EventTime) [Row]\r\n,RangePosition [Column]\r\nFROM \r\n{DataStructureParameterName} t\r\nGROUP BY \r\nDATEPART(HH,EventTime)\r\n,DATEPART(dd,EventTime) \r\n,RangePosition "
	}
]
```

A filter file - example "TopCurrencies.filter"

The filters are applied to the data imported, and allows us to apply a format to date time columns.

```json
[
	{
		"DataTypeName" : "ReportFormulaTest"
		,"FilterName" : "Last7DaysTopOnes"
		,"FilterIntialClause" : " WHERE validresult = 'true' "
		,"TextFilters" : 
		[
			{"FieldName" : "RangeGroup"
			,"TypeName" : "System.String"
			,"FilterValues" : ["BTC","ETH","BNB","ADA","XRP","DOGE","DOT","UNI","BCH",
			"LTC","SOL","LINK","WBTC","MATIC","ETC","THETA","XLM","ICP","DAI","VET","FIL","TRX","EOS"]
			,"TypeName" : "System.String"
			,"Template" : " AND [{FieldName}] in ({FilterValues}) "
			}
		]
		,"DateFilters" : 
		[
			{
			"FieldName" : "EventTime"
			,"WindowToDate" : "9999-12-31T00:00:00.00"
			,"PeriodsFromEnd" : -7
			,"DateFilterExpression" : " AND [{FieldName}] BETWEEN (SELECT  DATEADD(dd,{PeriodsFromEnd},'{WindowToDate}')) AND (SELECT '{WindowToDate}') "	
			}
		]
		,"FramePositions" : 
			[
				{
				"ColumnName" : "[Hourly]"
				,"Level" : 1
				,"Format" : " DATEPART([EventTime],hh) "
				}
				,
				{
				"ColumnName" : "[Week Day]"
				,"Level" : 2
				,"Format" : " DATEPART([EventTime],dw) "
				}
			]
	}
]
```


#### Steps involved
ProcessContainerQueryOutput, BuildContainerQueryOutput, DataProcessLoader


#### Expected inputs

##### 

DOCUMENTATION OUTSTANDING.

## DateOrdinalCalculationProcessor

This a more flexible mode for obtaining calculated/aggregated data over one or more sub timeframes relative to that now. For example, imagine we wanted to check  every ten minutes what the highest price was now, the highest price in the last ten minutes previous, the highest price previous to that, and the highest price ten minutes ahead, and the lowest price ten minutes

### Data Processing of files - contiguity
This is perhaps a little deep but when we save data in a database table, many RDMS will store these in pages/blocks, smaller data structures to aid with database performance. It would not make sense to have one large file with 10 billion records in there for most database systems. The Analytical Processor (AP) is different. We save data to files on disk, and we pull these through our processor. By the nature of the data we source, this data will probably be fragmented - perhaps we run a download for one hour, perhaps we run two? Perhaps we download 1000s of small files a day. The underlying library has a "Data Unioner" that can join data from multiple json files. However, we may end up with very large files that causes performance issues.

It would make sense that for large datasets, folders are partitioned into their respective range groups up-front. 
> Data 
Prices1.json
Prices2.json
Prices3.json
Prices4.json

	>> ADA
	>> BTC
	>> ETH
	>> LTC
	>> MASK

We may decide to have a separate mechanism to move and archive the files. We have applications that can help do this.

We will process each file individually. For the moment, the external process will take the data and manage what data exists within each file. For example, perhaps we create an hourly file of all the prices within that hour. Or a rolling 24 hour file? For example;
Day > 1
		(Hours)
			>> 0 Last 24hrs prices .json
			>> 1 Last 24hrs prices .json
			>> 2 Last 24hrs prices .json
			>> 3 Last 24hrs prices .json
			>> 4 Last 24hrs prices .json
			
It may make sense to do the following

Day > 1
Prices1.json
Prices2.json
Prices3.json
Prices4.json
		(Hours)
			>> 0.24.json
			>> 1.24.json
			>> 2.24.json
			>> 3.24.json
			>> 4.24.json
			

We may also consider storing this data to a database or document database and pushing data out to these folder structures using some form of publication service.

### How we will handle files now...

For the moment, we expect data to exist within a data folder root, we will write data back to that folder. We  We are likely to add more functionality to deal with different types of data items, whether they are contiguous or not. We will treat file as a single contiguous range and not attempt to do anything across multiple files.

## Build Data Array 

The build data array takes a json file which is a flat and rowset to convert a row which is comprised of one or more data items. This allows us to dynamically interpret data and avoid mapping to concrete objects which allows us to process data dynamically. For example, we can decide which fields we used to calculate volumes on based upon a time series column. It is for certain that this process is quite slow , there are ways to speed this up and this is scheduled for future development. As discussed previously it is a potential better approach to create the data to create the rows at the point of generating the json data. This would mean we would have to find the mechanism to decide whether we were loading json data that needs to be converted to data rows, or whether we are simply loading the data rows.

One reason for having the data format as data rows, was because it is perfectly possible that independent applications could develop could produce the data row collection independently of our Analyser Processor application.


```csharp
        public IEnumerable<Row> GetRows(string dataFileFullPath)
        {
            var json = reader.Read(dataFileFullPath);

            var rows = buildDataArray.GetRows(json);

            return rows;
        }
		
		    public class Row
    {
        public long Ordinal { get; set; }

        public IEnumerable<DataItem> DataItems { get; set; }


    }
	
	    public class DataItem
    {

        public string ColumnName { get; set; }

        public string Value { get; set; }

    }

```

## Configuration 

### Global.json
At the root of the Settings Path


Currently contains the following setting;

```json
{
	"millisecondIncrementer" : 1
}
```

### CalculationErrorDefault.json 
At the root of the Settings Path


A default value for a calculation

```json 

   [
    {

        "CalculationTypeName" : "Correlation",
        "ErrorDefaultValue" : 0

    }
	,
   {

        "CalculationTypeName" : "Count",
        "ErrorDefaultValue" : 0

    }
	,
	{

        "CalculationTypeName" : "FirstValue",
        "ErrorDefaultValue" : 0

    }
	,
	{

        "CalculationTypeName" : "LastValue",
        "ErrorDefaultValue" : 0

    }
	,
	{

        "CalculationTypeName" : "Max",
        "ErrorDefaultValue" : 0

    }
	,
	{

        "CalculationTypeName" : "Mean",
        "ErrorDefaultValue" : 0

    }
	,
	{

        "CalculationTypeName" : "Median",
        "ErrorDefaultValue" : 0

    }
	,
	{

        "CalculationTypeName" : "Min",
        "ErrorDefaultValue" : 0

    }
	,
	{

        "CalculationTypeName" : "Slope",
        "ErrorDefaultValue" : 0

    }
	,
	{

        "CalculationTypeName" : "StandardDeviation",
        "ErrorDefaultValue" : 0

    }
	,
	{

        "CalculationTypeName" : "Sum",
        "ErrorDefaultValue" : 0

    }
]

```

### FileTypeMapConfig.json

This contains all associated files used within the application. As you can see, there are a lot. We have made this so they can be configured to avoid hard-coding within the application but we strongly recommend never changing the details of the files.

```json
[
  {
    "ResourceType": "ActionData",
    "FileExtension": ".acd",
    "SearchPattern": "*.acd"
  },
  {
    "ResourceType": "ActionTimeColumnContainer",
    "FileExtension": ".actionTimeContainer",
    "SearchPattern": "*.actionTimeContainer"
  },
  {
    "ResourceType": "ActionTimeColumnContainerSource",
    "FileExtension": ".actionTimeColumnContainerSource",
    "SearchPattern": "*.actionTimeColumnContainerSource"
  },

  {
    "ResourceType": "ActionBucketFormulaTestResult",
    "FileExtension": ".calcresult",
    "SearchPattern": "*.calcresult"
  },
  {
    "ResourceType": "ActionDataBucketResult",
    "FileExtension": ".abr",
    "SearchPattern": "*.abr"
  },
  {
    "ResourceType": "ConfigChartResult",
    "FileExtension": ".configChartResult",
    "SearchPattern": "*.configChartResult"
  },
  {
    "ResourceType": "ConfigFormulaTest",
    "FileExtension": ".configFormulaTest",
    "SearchPattern": "*.configFormulaTest"
  },
  {
    "ResourceType": "ChartResult",
    "FileExtension": ".chartResult",
    "SearchPattern": "*.chartResult"
  },
  {
    "ResourceType": "Container",
    "FileExtension": ".container",
    "SearchPattern": "*.container"
  },
  {
    "ResourceType": "DataParameterHandler",
    "FileExtension": ".datahandler",
    "SearchPattern": "*.datahandler"
  },
  {
    "ResourceType": "DataSet",
    "FileExtension": ".json",
    "SearchPattern": "*.json"
  },
  {
    "ResourceType": "DataArray",
    "FileExtension": ".dataArray",
    "SearchPattern": "*.dataArray"
  },
  {
    "ResourceType": "GeneratedQuery",
    "FileExtension": ".generatedQuery",
    "SearchPattern": "*.generatedQuery"
  },
  {
    "ResourceType": "QueryHandlerBlock",
    "FileExtension": ".qryHandler",
    "SearchPattern": "*.qryHandler"
  },
  {
    "ResourceType": "FormulaAndTests",
    "FileExtension": ".formula",
    "SearchPattern": "*.formula"
  },


  {
    "ResourceType": "KeysRecommendation",
    "FileExtension": ".keyguide",
    "SearchPattern": "*.keyguide"
  },
  {
    "ResourceType": "ProcessMode",
    "FileExtension": ".mode",
    "SearchPattern": "*.mode"
  },
  {
    "ResourceType": "Query",
    "FileExtension": ".qry",
    "SearchPattern": "*.qry"
  },
  {
    "ResourceType": "QueryContainer",
    "FileExtension": ".queryContainer",
    "SearchPattern": "*.queryContainer"
  },
  {
    "ResourceType": "QueryFilter",
    "FileExtension": ".filter",
    "SearchPattern": "*.filter"
  },
  {
    "ResourceType": "DoneFile",
    "FileExtension": ".done",
    "SearchPattern": "*.done"
  },
  {
    "ResourceType": "ReportFormulaTest",
    "FileExtension": ".reportformulatest",
    "SearchPattern": "*.reportformulatest"
  },
  {
    "ResourceType": "RowSet",
    "FileExtension": ".rowset",
    "SearchPattern": "*.rowset"
  },
  {
    "ResourceType": "DateActionCalculation",
    "FileExtension": ".dateActionCalcConfig",
    "SearchPattern": "*.dateActionCalcConfig"
  },
  {
    "ResourceType": "RangeGroupDateResult",
    "FileExtension": ".rangeGroupDateResult",
    "SearchPattern": "*.rangeGroupDateResult"
  }
]
 

```



### Probable data folder arrangement

- C:\InfoRhino\Processing\CryptoStatto\Horizon\Hourly
- The Exe can be anywhere but we would have an instance per run so our log files will not conflict. We put it here. C:\InfoRhino\Processing\CryptoStatto\Horizon\Hourly\AnalyticalProcessor
- C:\InfoRhino\Processing\CryptoStatto\Horizon\Hourly\Data


```
