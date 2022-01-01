<!DOCTYPE html>
<head>
  <meta charset="utf-8">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.5/d3.min.js"></script>
  <style>
    body {
      position: fixed;
      top:0;left:0;bottom:0;right:0;
      margin:0
    }
    svg {
      width: 100%;
      height: 100%;
      background-color: #111;
    }
    .selected {
      fill: #d33f67;
    }
    .highlighted {
      stroke: #d33f67;
      stroke-width: 3;
    }
    .active {
      pointer-events: none;
    }
    .random {
      cursor: pointer;
    }
    .background {
      fill: #111;
    }
    .background.selected  {

    }
    .row {
      cursor: ew-resize;
    }
  </style>
</head>

<body>
  <svg></svg>

  <script>
    // the example we let you play with
    var example = [ 3, 2, 4, 2, 0, 2 ];
    //var example = [ 0, 1, 2, 3, 4, 0 ];
    var dimensions = example.length;
    var range = [0, 5]; // [0, 5)
    // the amount of vectors we want to compare with
    var num = 99;
    var randomData = d3.range(num).map(function(i) {
      return d3.range(dimensions).map(function() {
        return Math.floor(Math.random() * range[1])
      })
    })
    var selectedIndex = 3;
    var selectedData = randomData[selectedIndex]
    
    var svg = d3.select("svg")
    
    var randomg = svg.selectAll("g.random")
      .data(randomData)
    
    var cellSize = 8;
    var renter = randomg.enter().append("g").classed("random", true)
    var cols = 11
    renter.attr("transform", function(d,i) {
      var x = 10 + 83 * (i % cols);
      var y = 20 + 69 * Math.floor(i / cols);
      return "translate(" + [x, y] + ")"
    })
    .each(function(d,i) {
      var s = new strip()
      .data(d)
      .cellWidth(cellSize)
      .cellHeight(cellSize)
      .onChange(function(data) {
        rerender();
        updateExplanation();
      })
      .update(d3.select(this))
    })
    
    
    
    
    function cosineSimilarity(u, v) {
      // compute the distance between u and v
      var udotv = 0
      var umag = 0
      var vmag = 0
      for(var i = 0; i < u.length; i++) {
        udotv += u[i] * v[i]
        umag += u[i]*u[i]
        vmag += v[i]*v[i]
      }
      umag = Math.sqrt(umag)
      vmag = Math.sqrt(vmag)
      return udotv/(umag*vmag);
    }
    
  
    
    function strip() {
      //interactive vector editor
      var cellWidth = 20;
      var cellHeight = 20;
      var rows = 6;
      var cols = 5;
      var half = Math.floor(cols/2);
      var colors = d3.scale.ordinal()
        .domain(d3.range(rows))
        .range([
          "#e41a1c",
          "#377eb8",
          "#4daf4a",
          "#984ea3",
          "#ff7f00",
          "#ffff33"
        ])
      var data = [];
      var cb; // callback for onChange
      
      
      this.update = function(g) {        
        var mapping = d3.scale.linear()
          .domain([0, cols-1]) // based on max value
          .range([-(cols-1), 0])
      
        var container = g.append("g")
          .attr("transform", "translate(" + [cellWidth * cols, 0] + ")")
        
        var rowsg = container.selectAll("g.row")
          .data(d3.range(data.length))
        rowsg.enter().append("g").classed("row", true)
          .attr("transform", function(d) {
            var x = cellWidth * mapping(data[d]);
            var y = 0;
            return "translate(" + [x,y] + ")";
          })
        
        var cellsg = rowsg.selectAll("rect.cell")
          .data(function(d) { 
            return d3.range(cols).map(function(c) {
              return { c: c, r: d }
            })
          })
        
        cellsg.enter().append("rect").classed("cell", true)
        cellsg.attr({
          x: function(d) { return d.c * cellWidth },
          y: function(d) { return d.r * cellHeight },
          width: cellWidth,
          height: cellHeight,
          fill: function(d,i) { 
            var color = d3.hcl(colors(d.r))
            if(d.c < half) {
              return color.darker(1 - d.c/half)
            } else if(d.c > half) {
              return color.brighter(d.c/cols)
            } else {
              return color;
            }
          }
        })
        
        // We convert the relative mouse position to one of the potential
        // values in our array
        var mouseScale = d3.scale.linear()
        .domain([-cellWidth * cols/2, cellWidth/2 * (cols+1)])
        .range([0, cols-1])
        
        var drag = d3.behavior.drag()
        .on("drag", function(d) {
          var x = d3.event.x;
          var y = d3.event.y;
 
          var v = Math.floor(mouseScale(x))
          v = Math.max(0, Math.min(v, cols-1))
          
          if(data[d] === v) return;
          data[d] = v;
          rowsg.attr("transform", function(d) {
            var x = cellWidth * mapping(data[d]);
            var y = 0;
            return "translate(" + [x,y] + ")";
          })
          if(cb) cb(data);
        })
        rowsg.call(drag)
        
        return this;
      }
      
      this.data = function(val) {
        if(val) { data = val; return this; }
        return data;
      }
      this.cellWidth = function(val) {
        if(val) { cellWidth = val; return this; }
        return cellWidth;
      }
      this.cellHeight = function(val) {
        if(val) { cellHeight = val; return this; }
        return cellHeight;
      }
      this.onChange = function(val) {
        if(val) { cb = val; return this; }
        return val
      }
      return this;
    }
  </script>
</body>

<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');
  ga('create', 'UA-67666917-1', 'auto');
  ga('send', 'pageview');
</script>
