<head>
<title>Cambi gruppo Camera XVIII</title>
<link rel="stylesheet" href="style/style.css" type="text/css" media="screen" />
<link rel="stylesheet" type="text/css" href="//cloud.typography.com/7626174/696048/css/fonts.css" />
<link href='https://fonts.googleapis.com/css?family=Inconsolata' rel='stylesheet' type='text/css'>
</head>
<meta charset="utf-8">
<style>

.node {
  stroke-width: 1.5px;
}

</style>
<body>
   
<!-- Connecting with D3 library-->
<script src="https://d3js.org/d3.v3.min.js" charset="utf-8"></script>
  
<div id="main-wrapper">
<div id="sidebar">
<div id="title">Visualizzazione dei cambi di gruppo parlamentare alla Camera dei Deputati durante la XVIII legislatura</div>
<div id="current_time">2018-3-23</div>
<div id="speed">
<div id="block_container">
<button type="button" >
<div class="togglebutton slow current" data-val="slow">Lento</div>
</button>
<button type="button" >
<div class="togglebutton medium" data-val="medium">Medio</div>
</button>
<button type="button" >
<div class="togglebutton fast" data-val="fast">Veloce</div>
</button>
</div>
<div class="clr"></div>
</div>
<div id="note"></div>
</div>
<div id="chart"></div>
<div id="cite">
Autore: <a href="https://twitter.com/vi__enne" target="_blank" rel="noopener">V. Nicoletta</a> - 
Dati: <a href="https://parlamento18.openpolis.it/i-gruppi-in-parlamento/camera" target="_blank" rel="noopener">Openpolis</a> 
(licenza 
<a href="https://creativecommons.org/licenses/by-nc/4.0/" target="_blank" rel="noopener">CC-BY-NC-SA 4.0</a>) - 
<a href="https://vi-enne.github.io/cambiSenato18/" target="_blank" rel="noopener"><i>Cambi Senato clicca qui</i></a>
<br>
<br>
Basato sulla visualizzazione 
<a href="https://flowingdata.com/2015/12/15/a-day-in-the-life-of-americans/" target="_blank" rel="noopener">A Day in the Life of Americans</a> di 
<a href="https://flowingdata.com/about" target="_blank" rel="noopener">Nathan Yau</a>, e sul codice di 
<a href="https://gist.github.com/mohamedkhanafer/2f2ed4e2e7bff2c96481258675f88470" target="_blank" rel="noopener">Mohamed Khanafer</a> (licenza 
<a href="https://opensource.org/licenses/MIT" target="_blank" rel="noopener">MIT</a>)
</div>
<div class="clr"></div>
</div>
  <!-- Connecting with D3 library-->
<script src="https://d3js.org/d3.v3.min.js" charset="utf-8"></script>

  
  

<script>
var USER_SPEED = "slow";

var width = 900,
    height = 750,
	padding = 1,
	maxRadius = 3;
	// color = d3.scale.category10();
	
var sched_objs = [],
	curr_time = 0;

 
var act_codes = [
	{"index": "8", "short": "Forza Italia", "desc": "FI"},
	{"index": "9", "short": "Lega", "desc": "Lega"},
	{"index": "0", "short": "Fratelli d'Italia", "desc": "FdI"},
	{"index": "1", "short": "Misto", "desc": "Misto"},
	{"index": "5", "short": "Liberi e Uguali", "desc": "LeU"},
	{"index": "6", "short": "PD", "desc": "PD"},
	{"index": "4", "short": "Insieme Per il Futuro", "desc": "IpF"},
	{"index": "3", "short": "M5S", "desc": "M5S"},
	{"index": "2", "short": "Italia Viva", "desc": "IV"},
	{"index": "7", "short": "Coraggio Italia", "desc": "CI"},
	{"index": "10", "short": "Fuori Parlamento", "desc": "Out"},
];


var speeds = { "slow": 1000, "medium": 200, "fast": 20 };


var time_notes = [
	{ "start_time": 1, "stop_time": 18, "note": "Inizio XVIII legislatura della Repubblica Italiana" },
	{ "start_time": 19, "stop_time": 119, "note": "Formazione gruppo Liberi e Uguali" },
	{ "start_time": 540, "stop_time": 640, "note": "Nasce Italia Viva" },
	{ "start_time": 1064, "stop_time": 1159, "note": "Espulsione deputati M5S che non hanno votato la fiducia al governo Draghi" },
	{ "start_time": 1160, "stop_time": 1260, "note": "Nasce Coraggio Italia" },
	{ "start_time": 1550, "stop_time": 1650, "note": "Nasce Insieme Per il Futuro - Cessa di esistere Coraggio Italia" },
	{ "start_time": 1665, "stop_time": 2029, "note": "Fine XVIII legislatura della Repubblica Italiana" },

];
var notes_index = 0;


  
  
// Activity to put in center of circle arrangement
var center_act = "Out",
	center_pt = { "x": 380, "y": 365 };

  
  
  

// Coordinates for activities
var foci = {};
act_codes.forEach(function(code, i) {
	if (code.desc == center_act) {
		foci[code.index] = center_pt;
	} else {
		var theta = 2 * Math.PI / (act_codes.length-1);
		foci[code.index] = {x: 250 * Math.cos(i * theta) + 380, y: 250 * Math.sin(i * theta) + 365};
	}
});


// Start the SVG
var svg = d3.select("#chart").append("svg")
    .attr("width", width)
    .attr("height", height);
  
  
  
//Data loading   
d3.csv('https://gist.githubusercontent.com/vi-enne/7c489a661e8a5cdbfb6fb0efba0ea237/raw/a3990f41f623c67a8d875e0401c7b986ba4b170c/camera18', function(data) {
	data.forEach(function(d) {
		var day_array = d.day.split(",");
		var activities = [];
		for (var i=0; i < day_array.length; i++) {
			// Duration
			if (i % 2 == 1) {
				activities.push({'act': day_array[i-1], 'duration': +day_array[i]});
			}
		}
		sched_objs.push(activities);
	});
  
  
  
	// Used for percentages by time
	var act_counts = { "0": 0, "1": 0, "2": 0, "3": 0, "4": 0, "5": 0, "6": 0, "7": 0, "8": 0, "9": 0, "10": 0, "11": 0, "12": 0, "13": 0, "14": 0, "15": 0, "16": 0 };
	
  	// A node for each person's schedule
	var nodes = sched_objs.map(function(o,i) {
		var act = o[0].act;
		act_counts[act] += 1;
		var init_x = foci[act].x + Math.random();
		var init_y = foci[act].y + Math.random();
		return {
			act: act,
			radius: 3,
			x: init_x,
			y: init_y,
			color: color(act),
			moves: 0,
			next_move_time: o[0].duration,
			sched: o,
		}
	});
  
  
  	var force = d3.layout.force()
		.nodes(nodes)
		.size([width, height])
		// .links([])
		.gravity(0)
		.charge(0)
		.friction(.9)
		.on("tick", tick)
		.start();

	var circle = svg.selectAll("circle")
		.data(nodes)
	  .enter().append("circle")
		.attr("r", function(d) { return d.radius; })
		.style("fill", function(d) { return d.color; });
		// .call(force.drag);
	
	// Activity labels
	var label = svg.selectAll("text")
		.data(act_codes)
	  .enter().append("text")
		.attr("class", "actlabel")
		.attr("x", function(d, i) {
			if (d.desc == center_act) {
				return center_pt.x;
			} else {
				var theta = 2 * Math.PI / (act_codes.length-1);
				return 340 * Math.cos(i * theta)+380;
			}
			
		})
		.attr("y", function(d, i) {
			if (d.desc == center_act) {
				return center_pt.y;
			} else {
				var theta = 2 * Math.PI / (act_codes.length-1);
				return 340 * Math.sin(i * theta)+365;
			}
			
		});
  
  
  	label.append("tspan")
		.attr("x", function() { return d3.select(this.parentNode).attr("x"); })
		// .attr("dy", "1.3em")
		.attr("text-anchor", "middle")
		.text(function(d) {
			return d.short;
		});
	label.append("tspan")
		.attr("dy", "1.3em")
		.attr("x", function() { return d3.select(this.parentNode).attr("x"); })
		.attr("text-anchor", "middle")
		.attr("class", "actpct")
		.text(function(d) {
			return act_counts[d.index];
		});
	
  
  	// Update nodes based on activity and duration
	function timer() {
		d3.range(nodes.length).map(function(i) {
			var curr_node = nodes[i],
				curr_moves = curr_node.moves; 

			// Time to go to next activity
			if (curr_node.next_move_time == curr_time) {
				if (curr_node.moves == curr_node.sched.length-1) {
					curr_moves = 0;
				} else {
					curr_moves += 1;
				}
			
				// Subtract from current activity count
				act_counts[curr_node.act] -= 1;
			
				// Move on to next activity
				curr_node.act = curr_node.sched[ curr_moves ].act;
			
				// Add to new activity count
				act_counts[curr_node.act] += 1;
			
				curr_node.moves = curr_moves;
				curr_node.cx = foci[curr_node.act].x;
				curr_node.cy = foci[curr_node.act].y;
			
				nodes[i].next_move_time += nodes[i].sched[ curr_node.moves ].duration;
			}

		});

		force.resume();
		curr_time += 1;

    
 		// Update percentages
		label.selectAll("tspan.actpct")
			.text(function(d) {
				// return readablePercent(act_counts[d.index]);
				return act_counts[d.index];
			});
	
		// Update time
		var end_time = 2030
		var true_time = curr_time % end_time;
		d3.select("#current_time").text(daysToDate(true_time));
		
		// Update notes
		// var true_time = curr_time % 1440;
		if (true_time == time_notes[notes_index].start_time) {
			d3.select("#note")
				.style("top", "0px")
			  .transition()
				.duration(600)
				.style("top", "20px")
				.style("color", "#000000")
				.text(time_notes[notes_index].note);
		} 
	
		// Make note disappear at the end.
		else if (true_time == time_notes[notes_index].stop_time) {
			
			d3.select("#note").transition()
				.duration(1000)
				.style("top", "300px")
				.style("color", "#ffffff");
				
			notes_index += 1;
			if (notes_index == time_notes.length) {
				notes_index = 0;
			}
		}
		
		
		setTimeout(timer, speeds[USER_SPEED]);
	}
	setTimeout(timer, speeds[USER_SPEED]);
	
  
	function tick(e) {
	  var k = 0.04 * e.alpha;
  
	  // Push nodes toward their designated focus.
	  nodes.forEach(function(o, i) {
		var curr_act = o.act;
		
		// Make sleep more sluggish moving.
		if (curr_act == "0") {
			var damper = 0.6;
		} else {
			var damper = 1;
		}
		o.color = color(curr_act);
	    o.y += (foci[curr_act].y - o.y) * k * damper;
	    o.x += (foci[curr_act].x - o.x) * k * damper;
	  });

	  circle
	  	  .each(collide(.5))
	  	  .style("fill", function(d) { return d.color; })
	      .attr("cx", function(d) { return d.x; })
	      .attr("cy", function(d) { return d.y; });
	}

	// Resolve collisions between nodes.
	function collide(alpha) {
	  var quadtree = d3.geom.quadtree(nodes);
	  return function(d) {
	    var r = d.radius + maxRadius + padding,
	        nx1 = d.x - r,
	        nx2 = d.x + r,
	        ny1 = d.y - r,
	        ny2 = d.y + r;
	    quadtree.visit(function(quad, x1, y1, x2, y2) {
	      if (quad.point && (quad.point !== d)) {
	        var x = d.x - quad.point.x,
	            y = d.y - quad.point.y,
	            l = Math.sqrt(x * x + y * y),
	            r = d.radius + quad.point.radius + (d.act !== quad.point.act) * padding;
	        if (l < r) {
	          l = (l - r) / l * alpha;
	          d.x -= x *= l;
	          d.y -= y *= l;
	          quad.point.x += x;
	          quad.point.y += y;
	        }
	      }
	      return x1 > nx2 || x2 < nx1 || y1 > ny2 || y2 < ny1;
	    });
	  };
	}

	
	// Speed toggle
	d3.selectAll(".togglebutton")
      .on("click", function() {
        if (d3.select(this).attr("data-val") == "slow") {
            d3.select(".slow").classed("current", true);
			d3.select(".medium").classed("current", false);
            d3.select(".fast").classed("current", false);
        } else if (d3.select(this).attr("data-val") == "medium") {
            d3.select(".slow").classed("current", false);
			d3.select(".medium").classed("current", true);
            d3.select(".fast").classed("current", false);
        } 
		else {
            d3.select(".slow").classed("current", false);
			d3.select(".medium").classed("current", false);
			d3.select(".fast").classed("current", true);
        }
		
		USER_SPEED = d3.select(this).attr("data-val");
    });
}); // @end d3.tsv

function color(activity) {
	
	var colorByActivity = {
		"0": "#1a237e",
		"1": "#78909c",
		"2": "#ab47bc",
		"3": "#ffeb3b",
		"4": "#1b5e20",
		"5": "#b71c1c",
		"6": "#f44336",
		"7": "#f06292",
		"8": "#0d47a1",
		"9": "#303f9f",
		"10": "#d7ccc8",
	}
	
	return colorByActivity[activity];
	
}



// Output readable percent based on count.
function readablePercent(n) {
	var pct = 100 * n / 1000;
	if (pct < 1 && pct > 0) {
		pct = "<1%";
	} else {
		pct = Math.round(pct) + "%";
	}
	
	return pct;
}


// Day. Data is days from 2018-03-23.
function daysToDate(m) {
    m = Math.min(m,1665);
    var d = new Date("2018-03-23");
    d.setDate(d.getDate()+m);
    var y = d.getFullYear();
    var da = d.getDate();
    var mo = d.getMonth() + 1;
    return (y+'-'+mo+'-'+da);
}


</script>
