---
#layout: post
title: The overview and nitty-gritty bit of Tableau
description: >
  A beginner guide to familiarize with the fundamental concepts and development workflow of Tableau. 
author: author1
comments: true
---

<link rel="stylesheet" type="text/css" href="/assets/css/pill.css">


To facilitate the discovery of data insights or the monitoring of a model forecast, Tableau is one of the options that can help you quickly prototype an interactive dashboard. 
While it is not free, if your company does procure its license, it's a no-brainer to pick up this skill due to its vast usefulness. 

As a Tableau beginner myself, despite everything is one google away, I often caught myself scratching my head when I first started due to the lack of understanding of what Tableau has to offer, hence the right keyword does not pop up at first try.
Given that, I believe a run-down of Tableau's essential concepts may benefit readers in approaching their first dashboard systematically and steer them in the right direction. Hence the writing of this topic.

A dashboard development typically consists of 3 stages: **Connect**, **Analyze**, and **Share**. The post is organized in a similar manner, plus an introduction of Tableau's interface and high emphasis on Analyze section because most development works happen there. It's noteworthy that there is a shift in calculating mindset, should the developers mostly use Python in their daily job. The impression that there is little room for sophisticated logic implementation can be clear up when you adopt the mindset of column-based calculation, which is much similar to working with Excel. Keep in mind though, at the end of the day, Tableau is a visualization tool and not a data processing engine. 

Once reaching the end of this post, my objective is the readers are equipped with the ability to sketch your dashboard implementation in clear and deliverable steps, and know what to look up to implement your dashboard at details level. You may find some part not relevant at first and do feel free to skip it. What matters is capturing the flow of a dashboard life cycle!

- [Navigation](#navigation)
- [Connect](#connect)
- [Analyze](#analyze)
  - [Workspace](#workspace)

## Navigation

Launching the software for the first time, you will find yourself in the **Start page**. Here you can either connect to a data source (**Data Source page**), or jump right into the **Workspace** to work on an existing workbook.
The two formers can switch back and forth using the **Sheet tabs** on the window's bottom left. 

![Fig01](/assets/blog/2024-02-03/tableau-page-navigation.png){:data-width="1440" data-height="836"}
Fig. 1. The navigation flow between three pages developers commonly work on: <a href="https://help.tableau.com/current/pro/desktop/en-us/environment_startpage.htm">Start page</a>, <a href="https://help.tableau.com/current/pro/desktop/en-us/environment_datasource_page.htm">Data Source page</a> and <a href="https://help.tableau.com/current/pro/desktop/en-us/environment_workspace.htm">Workspace</a>. Source: https://help.tableau.com/ 
{:.figure}


## Connect

Tableau can connect to a variety of data sources: file-based ( e.g. .csv, .json, Excel, text file), SQL data bases, Hadoop-compatible file format such as Parquet, and Hive table, to name but a few. 
For starter, it's easiest to convert a mock data into .csv file and connect to it. 
Once the drafted workbook functions well, you can easily switch to the original data source to display live data.

> **Tip \| Replace data source**: assume you have an alternative dataset with the same schema, you can switch between them without rebuilding the workbook from scratch by right click on a field in dataset's attributes, select "Replace References...". If there is a difference in column's naming in some, you have to provide the 1-1 mapping for those.

Figure 2 captures the overall look and basic interactive components to whom you need to pay attention.

1. Connections displays the selected Excel data source; its multiple sheets are too displayed if applicable.
2. Database icon allows to switch the view between datasets.
3. **Schema** - the column name and its data type - of the selected dataset, with 1000 rows of examples. You can rename the field name. You will notice icon like <span style="color: blue; font-weight: bold;">Abc</span> or <span style="color: green; font-weight: bold;">#</span>. <span style="color: blue; font-weight: bold;">Blue icon</span> indicates the **field** (column) is discrete, say, name of countries in flight records. Similarly, <span style="color: green; font-weight: bold;">Green icon</span> indicates continuous field, say, the transaction price in supermarket records. 
4. **Extract mode**: how data is retrieved from source. A Live connection to track and reflect all the latest change, which is often favorable when the dashboard goes live. 
I suggest to opt for Extract mode during development so you can trigger the data retrieval on command, hence dealing with static data. 
Dashboard that I've dealt with so far are viewed on a daily basis, so I set Extract with a CRON job (time-based scheduler) because I found this setup is stabler.

> **Tip \| Refresh and Extract**: Extract refers to retrieving data, and Refresh the reloading of the whole workbook. If your dataset is updated, you should ensure the Extract is triggered, either manually or live, otherwise the Refresh alone will not show that change. 

![Fig02](/assets/blog/2024-02-03/tableau-datasource-page.png){:data-width="1440" data-height="836"}
Fig. 2. Data Source page's interface. Source: https://interworks.com/blog/rcurtis/2018/01/31/tableau-extended-data-connection-page/ 
{:.figure}

Besides the above, you can **join two datasets** to create a more meaningful third one. 
You can create a user-defined calculation or copy of another field, displayed with an equal sign (=) preceding an icon. 
More on this topic to be discussed in Analyze section, but I find the **Custom Split** feature is regularly used to separate string into new columns, say, splitting full name into family and given name.
Lastly, you can save data source in two formats. The **.tds file** does not contain data from underlying source, but your modification and connection info you have added. 
The **.tdsx file** contains the info of .tds file, plus a local copy of the data, in case your colleagues do not have access permission to such data.


## Analyze

### Workspace

Workspace is a canvas and Fields from **Data Pane** are elements to make up charts. 
By drag-and-drop them directly into the **Worksheet**, or indirectly into **Columns/Rows Shelf** and **Marks Card**, you create a chart of your desire. 
**Show Me** button on the far right will prompt the best practice charts that suit the provided elements.
At the same time, Tableau gives you the option to customize charts with Marks Card and **Filter Shelf**.
Other common components are depicted in Figure 3.

> **Tip \| Right Click**: to look for an operation to interact with a certain component in the workspace, the right click quite often have such action integrated in the option list already. 

![Fig03](/assets/blog/2024-02-03/tableau-worksplace-page.png){:data-width="1440" data-height="836"}
Fig. 3. Workspace's interface. Source: https://www.reddit.com/r/tableau/comments/fsajiw/pls_help_with_pokemon_scatter_plot/
{:.figure}

### Data Pane

Tableau interprets dataset's fields into:

|Field|Definition
-|-
Dimension fields| categorical variables containing qualitative information such as text and dates. This typically set the level of aggregation for numeric data.
Measure fields| numerical variables containing numbers which can be aggregated. 
Calculated fields| new variable that is the transformation of the existing fields, similar to formula in Excel and also depicted with an equal sign (=).
Sets| represent subset of data that meets certain condition based on existing fields. Effective to view, highlight, monitor key data points; combine fields; enables displaying ins and outs of a set, ideal for point-based chart for anomaly detection. 
Parameters| variables that can act as constant values, hence allows to control the view in some way. 

For example, I have a dataset of [hourly traffic records](https://www.kaggle.com/datasets/fedesoriano/traffic-prediction-dataset). Its schema could be interpreted as follows:

|Column Name|Data type|Description
-|-|-
`Date`|Dimension field| timestamp in format DDMMYYYY.
`Vehicles`|Measure field| number of observed vehicles.
`Max(Vehicles)`|Calculated field| maximum value of `Vehicles` in an hour.
`Vehicles(<1000)`|Set| set of data points whose `Vehicles` less than 1000.
`SelectedDate`|Parameters| could be today or a date of interest.

It's possible to convert dimension field to measure field, and vice versa, given appropriate context. 
On one hand, if measure field has a fix set of value, say, a `Gender` field in which Male=0 and Female=1, that can be directly convert into dimension.
On the other hand, you cannot directly convert a dimension field with text values to measure; still, you can use calculated field to perform mathematical operations, aggregations or other transformation to effectively turning it into a measure. 
For example, a `CharacterCount` derived from `UserReview` in a product review dataset.

To structure fields in a systematic manner, you can also arrange them in form of a **Hierarchy** to capture their relationship.
For example, Tableau typically captures `Date` as a hierarchy representing date parts (year, quarter, month, day) at different aggregation level.
User can flexibly treat them as continuous, say, axe of a time series, or discrete variable, say, a set of year-to-year trend per quarter, to suit their own need.
For 

> **Tip \| Date part & Date value**: the two properties of `Date` determine it is discrete or continuous, which can be a bit confusing at first. Check out [this page](https://theinformationlab.nl/2022/07/11/tableau-date-parts-vs-date-values/) for detailed explanation.



### Chart Creation

In essence, you drag fields from Data Pane to Columns & Rows Shelf to form x-axis and y-axis. 
Depends on the selected field is discrete or continuous, and its aggregation level, the chart is populated correspondingly.
Marks card let you tweak the appearance of data points in the chart, in terms of shape, color, size, label, detail, and tooltip.
To have more control on displayed data, you can add Filter to conditioned data point, say, data in the recent 2 weeks.

Alternatively, you can reverse-engineer by looking up the chart of your interest then figure out the right combination.
Show Me is very helpful to suggest the best practice chart given your choice of fields.
Interestingly, across the dashboards I have worked with, the most common chart is in fact very basic: **Crosstab chart**, or Pivot table or Text table.
This Excel-like table let you drill down to row-level detail, enabling a zoom-in feature to support other charts, or itself provide a practical chart for studying the data at the beginning.

![Fig04](/assets/blog/2024-02-03/tableau-chart-creation-example.png){:data-width="1440" data-height="836"}
Fig. 4. Illustration of how chart is created differently based on Fields dragged into Rows and Columns Shelf, and their continuous or discrete nature.
{:.figure}

When combine two fields into an axis, or dual axis view, you will notice that newly Tableau-generated fields called **Measure Values** (dimensions) and **Measure Names** are added into the view, under Marks card and sometimes Filter. 
These fields serve as a container for measures in the chart, so you can group chart values or label measures with their values.
Dual axis view does not automatically sync the scale between the two axises, so you need to right click to ensure the property is set.
To make your chart more story-like, make use of tool tips to reword the displayed text to be more natural, rather than a mere list of attributes.
Filters are straightforward to limit the chart value range. 


### Analytics Pane

If you have a chart created, Analytics Pane has options to quickly add pre-defined components such as constant and average lines, medians with quartiles, box plots, and totals... to the chart.
Trend and forecasting model for line charts can also be found, say, trend lines, forecasting, average distribution band.

### Computational Mindset

More than often, the existing data does not contain the key measure you want to display in the chart.
Your first instinct should see if this is better to have a new field added in the original data source, and double check if **Table Calculation** can provides what you want, say, pre-computed aggregation, running total, etc... dynamically response to your visualization's layout and partitioning.
However, if you opt for a more customized approach, **Calculate Field** is another solution and the focus of this section.

For someone using Python most of the time, I find it difficult at first to adapt to Tableau's computational mindset. 
Instead of writing a block of code to transform the data as will, I should handle it similarly to an Excel file.
In order to compute a sophisticated column, you break down the logic into step-by-step and use multiple columns to capture them. 
This translates into multiple fields in Tableau and provides a complete Excel-like experience with formula.

There are certain practices I learnt along the way:
- Be cautious of NULL value. Crosstab will not show data row whose value is NULL. Can use ZN function to replace NULL with zero (0).
- If you refer to a constant, makes use of Parameter.
- **Level of Detail** (LOD) expressions define the granularity or aggregation of the data in your view. It's useful to avoid potential misleading when aggregated. Keywords: Fixed, Include, Exclude. 
- Performance: Calculated fields are processed on database side, while Table calculation are processed locally.


### Troubleshooting

Firstly, **View Data** is your friend.
For visualization element that you are unsure about, highlight to select and right click View data to check the underlying data. 
You can view the data constructing the chart, or the pre-filtered data, or even export to file for a closer look.

Secondly, sometimes the bug lurking in the Calculated Fields' logic. By clicking Edit, you can examine the formula to debug.
There was one instance where I didn't have any filter turned on, but a data row was missing. Turning out, the calculated field didn't return value for that particular row due to a value check condition.

### Dashboard & Worksheet

Dashboard: make use of grid so think are well defined.
worksheets


how to jump to worksheet from dashboard. why we typically want to hide worksheet.
Application Terminology: Layout tab, one can say one chart is one worksheet, put them together either dashboard or story
navigate between worksheet and dashboard. 

A dashbaord is a collection of worksheets and supporting information shown in a single view. display all views at once

If you have two charts sharing the same filters, to make your space 
Sync filter




### Layout
the views in dashboard aer connected to worksheets they represent. When you make change to a view, either on the worksheet directly or on its dashboard representation.

tiled components: you can set sheets and objects to tiled. Tiled components are arranged in a grid consisting of layout containers. 
fitting a view to its container, makes use of the layout to know the order. Fit -> Entire view, standard.... Distribute if multiple sheets or objects is placed in a horizontal or vertical layout container.

Floating components: set to floating, move or resize. 

------------




calculated field when it applies row wise or for each record


interact with chart zone to make chart
details to disaggregate - level of details
highlighter - similar to filter

book 2
context filter can help.performnce because filter only applies on the resulting tables

Build your own: Drag fields to Columns, Rows, and Mark cards (change color, sizes, marks, click on the object of interest, determine the chart). Equivalent to Python.

Mindset when compute, transformation. If too complex then you should do it on the data source side (shift left). column steps by step.

Crosstabs: most common representation of daa, useful for viewing specific numeric values. 1 dim on Columns shelf, 1 dim on Rows shelf, a measure to Text on Marks card. on Analysis, can enable Show Row Grand Totals, Column.
Changing the aggregation of measures. 


Analyze: right click the field on Marks card, then click Show Highlighter, or Using tooltip - tooltip selection is on.  

calculated fields: when your underlying data does not contain all of the values you need for your analysis.
- Fields: data source fields and calculated fields
- functions, operators, 
- parameters: placeholder variables that can be inserted into calculations to replace constant values. comments
we have predefined calculations (Table Calculations, Row and Column Totals on Analysis menu)

table of example of different combinations to create different chart. Link to gallery to decipher or make it by themselves



Ctrl+C from another chart over. 



a parameter is a value users can change when interacting with  a view, it allows users to have input control over the visualization. this is not specific to data source or worksheet, so the defined value can be used acorss the entire workbook and across disparate sources.
FIXED [State]: Sum[Sales]

It has the models: trend line, forecasting, reference line, band

layout: book 2, 136




LoD: book3, 46
Fixed expresiosin associate an agrgegate expression (SUM, AVG) to a particular dimension or dimensions regarding of what dimensions and values are currently in the view. 
3 types of calcuation: book 3, 57


tableau: diff between refresh and extract

List out most used actions:
- design layout for dashboard 
- get data from source
- publish workbook
- additional 
- data query, data update, CRON job with subscription
- trouble shooting: View Data 


## Share
Where to put the file format? Share section I think?

## Summary
by able to name thíng correctly, the problem is half solved

what is the extent that the software is câpble of

---