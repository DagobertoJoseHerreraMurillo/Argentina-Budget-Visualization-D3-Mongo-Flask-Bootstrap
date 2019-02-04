# Argentina Budget Visualization using MongoDB and D3.js/DC.js

![](https://github.com/sergiers3/Argentina-Budget-Visualization-D3-Mongo-Flask-Bootstrap/blob/master/gif.gif?raw=true)

This project contains a visualization of Argentina's budget expenditure from 2015 to 2019. We describe the ”Argentinian National Budget 2015-2019” application, which is designed so that through it any citizen can visualize the structure of the national budget, paying special attention to the functions of spending (social services, defense, debt, etc.). The principles of visualization presented here apply to the visualization of public accounts in any country in the world that uses a multidimensional financial model.

Our application structures the visualization of a budget information dataset linearly, following the concepts of the profile page and the inverted pyramid (writing style is characterized by putting the most relevant information at the beginning to grab the reader’s attention from the beginning). As already mentioned, the multidimensional nature of the budget information causes different stakeholders to focus their attention on different questions. This is vital because there may be multiple possible design structures and the decision on which to use depends on the profile of the main user. Our analysis assumes that the main stakeholder is a citizen who has a special interest in understanding the purpose and function of the expense. That is why the head of our application focuses first on addressing this issue. From this central question, in the successive views you can explore elements of other dimensions in the form of basic ’w-questions’ (When? Where? Who? What?).



It is implemented using [DC.js](https://github.com/dc-js/dc.js), is a javascript charting library that uses [D3.js](https://d3js.org/) with native crossfilter support.

In the [report](https://github.com/sergiers3/Argentina-Budget-Visualization-D3-Mongo-Flask-Bootstrap/blob/master/doc/Report.pdf) you can find the reasons for our decisions.


## Instalation:

Download the repository and install the Requirements. Run:

```python
pip install -r requirements.txt
```

After installing [MongoDB](https://www.mongodb.com/es), you can download the dataset from the [original source](https://www.presupuestoabierto.gob.ar/sici/), but we are using a subset of the data (~10k).

Then, you need to import the dataset to the document store. See how it is done in windows:

```
mongoimport -d donorschoos -c projects --type csv --headerline --file C:\XXXXXXXXXX\sampledata.csv
```

Finally, run the flask app using the terminal:

```
python app.py
```

Dont forget to chek the [Flask Documentation](http://flask.pocoo.org/docs/1.0/) for more info about defining the ports and routes.


## Where magic happens:

The heart of the visualization is inside the [Dashboard.js](https://github.com/sergiers3/Argentina-Budget-Visualization-D3-Mongo-Flask-Bootstrap/blob/master/static/js/Dashboard.js) class. Check it out:



```Javascript
	////////////
	// CHARTS //
	////////////

	// total spent
	totalSpentChart
		.formatNumber(d3.format("d"))
		.valueAccessor(function(d){return d; })
		.group(budgetGroup)
		.formatNumber(d3.format("$.3s"));

	// Time line chart
	dateChart
		.height(220)
		.margins({top: 10, right: 50, bottom: 30, left: 50})
		.dimension(datePosted)
		.group(groupByDate)
		.renderArea(true)
		.transitionDuration(500)
		.x(d3.time.scale().domain([minDate, maxDate]))
		.elasticY(true)
		.renderHorizontalGridLines(true)
    	.renderVerticalGridLines(true)
		.xAxisLabel("Year")
		.yAxis().ticks(6);


	// Source barchart
	sourceBarChart
		.height(300)
        .dimension(budgetMoney)
        .group(sourceGroup)
        .ordering(function(d) { return -d.value })
        .xAxis().ticks(4);

    // Institution barchart
	institutionChart
		.height(1000)
        .dimension(budgetMoney)
        .group(institutionGroup)
        .rowsCap(20)
        .ordinalColors(['lightblue'])
        .ordering(function(d) { return -d.value })
        .xAxis().ticks(4);

  
  	// Funding piechart
	fundingChart
		.height(300)
		.radius(140)
		.innerRadius(40)
		.transitionDuration(1000)
		.dimension(budgetMoney)
		.group(caracterGroup)
		.legend(dc.legend());

	// Bubblechart
    bubbleChart
		.width(1000)
        .height(600)
    	.dimension(type) 
        .group(budgetReduced)
		.colorAccessor(function(d) {
            return d.value.col;
        })  
        // X axis
        .keyAccessor(function(d) {return d.value.col;})
        // Y axis
        .valueAccessor(function(d) {return d.value.y;})
        .radiusValueAccessor(function(d) {return d.value.sum;})
        .x(d3.scale.linear().domain([-100, 1000]))
        .y(d3.scale.linear().domain([-100, 1000]))
        // set radius scale
        .r(d3.scale.linear().domain([0, 300000]))
		// (optional) render horizontal grid lines, :default=false
        .renderHorizontalGridLines(false)
        // (optional) render vertical grid lines, :default=false
        .renderVerticalGridLines(false) 
        .renderLabel(true)
        .renderTitle(false)
        .elasticX(true)
        .elasticY(true)
        .yAxisPadding(2500)
        .xAxisPadding(1)
        .maxBubbleRelativeSize(0.4)
        .margins({top: 10, right: 50, bottom: 30, left: 40});

    // map
	arMapChart.width(1000)
		.height(600)
		.dimension(state)
		.group(totalDonationsByState)
		.colors(["#E2F2FF", "#C4E4FF", "#9ED2FF", "#81C5FF", "#6BBAFF", "#51AEFF", "#36A2FF", "#1E96FF", "#0089FF", "#0061B5"])
		.colorDomain([0, max_state])
		.overlayGeoJson(statesJson["features"], "state", function (d) {
			return d.properties.name;
		})
    	.projection(d3.geo.transverseMercator()
                     .center([2.5, -38.5])
                     .rotate([66, 0])
                     .scale((500 * 56.5) / 33)
                     .translate([(450), (300)])
    		)
		.title(function (p) {
			return "State: " + p["key"]
					+ "\n"
					+ "Total Donations: " + Math.round(p["value"]) + " $";
		});
  
	// Table of atomic data
	  dataTable.width(1000).height(400)
	    .dimension(datePosted)
		.group(function(d) { return ""
		 })
		.size(100)
	    .columns([
	      function(d) { return d.school_county; },
	      function(d) { return d.school_kipp; },
	      function(d) { return d.total_price_excluding_optional_support; }
	    ])
	    .sortBy(function(d){ return d.total_price_excluding_optional_support; })
	    .order(d3.descending);

    dc.renderAll();
};
```



