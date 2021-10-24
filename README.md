<!DOCTYPE html>
<meta charset="utf-8">

<style>
  h3 {text-align: center;}
  .container{  
  text-align: center;  
  }  
</style>
<!-- Legend --> 
<div id="legend1" style="display:relative;top:450px;left: 100px;">
</div>
<!-- Legend title --> 
<div class = "legend" style ="position:relative; top:225px; padding-left: 100px;">
<strong>Legend</strong></div>

<!-- Load d3.js -->
<script src="https://d3js.org/d3.v4.js"></script>
<script src="https://d3js.org/d3-scale-chromatic.v1.min.js"></script>

<h3 class = "title">Average Monthly Minimum Temperature of City X</h3>

<!-- Maximum button and minimum button -->
<!-- Create the buttons -->
<div class="container">  
<input type = "button" onclick="update('min_temperature')" class = "min" value = "min" >
<input type = "button" onclick="update('max_temperature')" class = "max" value = "max" ><br>
</div>

<!-- Create a div where the graph will take place -->
<div id="my_dataviz" style="width:800px; margin:0 auto;">

<script>
    //Change title to Average Maximum Monthly Temperature of City X
    function TitleMin(){
    newTitle = "Average Minimum Monthly Temperature of City X";
    document.querySelector('h3').textContent = newTitle;
    }

    //Change title to Average Minimum Monthly Temperature of City X
    function TitleMax(){
    newTitle = "Average Maximum Monthly Temperature of City X";
    document.querySelector('h3').textContent = newTitle;
  }

    // set the dimensions and margins of the graph
    var margin = {top: 30, right: 30, bottom: 30, left: 30},
      width = 800 - margin.left - margin.right,
      height = 550 - margin.top - margin.bottom;
    
    // append the svg object to the body of the page
    var svg = d3.select("#my_dataviz")
    .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
    .append("g")
      .attr("transform",
            "translate(" + margin.left + "," + margin.top + ")");
    
    // Labels of row and columns
    var myGroups = ["1997", "1998", "1999", "2000", "2001", "2002", "2003", "2004", "2005", "2006","2007","2008","2009","2010","2011","2012",
                    "2013","2014","2015","2016","2017"]
    var myVars = ["01", "02", "03", "04", "05", "06", "07", "08", "09", "10","11","12"]
  
    // Build X scales and axis
    var x = d3.scaleBand()
      .range([ 0, width ])
      .domain(myGroups)
      .padding(0.05);

    svg.append("g")
      .attr("transform", "translate(0," + height + ")")
      .call(d3.axisBottom(x))
    
    // Build Y scales and axis:
    var y = d3.scaleBand()
      .range([0,height])
      .domain(myVars)
      .padding(0.05);

    var y_axis = d3.axisLeft().scale(y)
    .tickValues(myVars)
    .tickFormat((d, i) => ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct","Nov","Dec"][i]); 

    svg.append("g")
      .call(y_axis)
    
    // Build color scale
    var myColor = d3.scaleSequential()
     .interpolator(d3.interpolateOranges)
     .domain([10,35]);

    // create a tooltip
    var tooltip = d3.select("#my_dataviz")
      .append("div")
      .style("opacity", 0)
      .attr("class", "tooltip")
      .style("background-color", "white")
      .style("border", "solid")
      .style("border-width", "2px")
      .style("border-radius", "5px")
      .style("padding", "5px")

    // Three function that change the tooltip when user hover / move / leave a cell
    var mouseover = function(d) {
      tooltip.style("opacity", 1)
      d3.select(this)
      .style("stroke", "black")
      .style("opacity", 1)
    }

    // retrieve month for tooltip
    function getCurrentMonthName(dt){
      monthNamelist = [ "January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December" ];
    return monthNamelist[dt-1];
    };

    var mouseleave = function(d) {
      tooltip.style("opacity", 0)
      d3.select(this)
      .style("stroke", "none")
    }

     // Add X axis --> it is a date format
     var xline = d3.scaleTime()
      .domain([1.,30])
      .range([ 1,30 ]);

    // Add Y axis
    var yline = d3.scaleLinear()
      .domain([15,35])
      .range([15,35]);

  function update(selectTemp){
    
      //Read the data
      d3.csv("CityX_temp_monthly_Cleaned.csv", function(data) {
        var mousemove = function(d) {
          tooltip
            .html("Date: "+ getCurrentMonthName(d.month) + " " + d.year + " ; " +"Maximum Temperature: " + d[selectTemp] + " degrees")
            .style("left", (d3.mouse(this)[0]+70) + "px")
            .style("top", (d3.mouse(this)[1]) + "px")
        }
      
        // Select all matching elements
        svg.selectAll()
            .data(data, function(d) {return d.year+':'+d.month;})
            .enter()
            .append("rect")
            .attr("x", function(d) { return x(d.year) })
            .attr("y", function(d) { return y(d.month) })
            .attr("width", x.bandwidth() )
            .attr("height", y.bandwidth() )
            .style("fill", function(d) { return myColor(d[selectTemp])} )
            .on("mouseover", mouseover)
      .on("mousemove", mousemove)
      .on("mouseover", mouseover)
      .on("mouseleave", mouseleave)
      })
  }

 // Call the legend function
 continuous("#legend1", myColor);

// create continuous color legend
function continuous(selector_id, colorscale) {
  var legendheight = 200,
      legendwidth = 80,
      margin = {top: 10, right: 60, bottom: 10, left: 2};

  var canvas = d3.select(selector_id)
    .style("height", legendheight + "px")
    .style("width", legendwidth + "px")
    .style("position", "relative")
    .append("canvas")
    .attr("height", legendheight - margin.top - margin.bottom)
    .attr("width", 1)
    .style("height", (legendheight - margin.top - margin.bottom) + "px")
    .style("width", (legendwidth - margin.left - margin.right) + "px")
    .style("border", "1px solid #000")
    .style("position", "absolute")
    .style("top", (margin.top) + "px")
    .style("left", (margin.left) + "px")
    .node();

  var ctx = canvas.getContext("2d");

  var legendscale = d3.scaleLinear()
    .range([1, legendheight - margin.top - margin.bottom])
    .domain(colorscale.domain());

  var image = ctx.createImageData(1, legendheight);
  d3.range(legendheight).forEach(function(i) {
    var c = d3.rgb(colorscale(legendscale.invert(i)));
    image.data[4*i] = c.r;
    image.data[4*i + 1] = c.g;
    image.data[4*i + 2] = c.b;
    image.data[4*i + 3] = 255;
  });
  ctx.putImageData(image, 0, 0);

  // Create legend axis
  var legendaxis = d3.axisRight()
    .scale(legendscale)
    .tickSize(6)
    .ticks(6);

  // Create svg element
  var svg = d3.select(selector_id)
    .append("svg")
    .attr("height", (legendheight) + "px")
    .attr("width", (legendwidth) + "px")
    .style("position", "absolute")
    .style("left", "0px")
    .style("top", "0px")

  // Create legend element
  svg
    .append("g")
    .attr("class", "axis")
    .attr("transform", "translate(" + (legendwidth - margin.left - margin.right + 3) + "," + (margin.top) + ")")
    .call(legendaxis);
};
    
  // Initialize min_temperature and preload data of heatmap
  update('min_temperature')
    </script>

</div>
  <br>

  
