	// round corners
	Chart.pluginService.register({

		afterDraw: function (chart) {
    console.log(chart);
			if (chart.config.options.elements.arc.roundedCornersFor !== undefined) {
      	var datasets = chart.chart.config.data.datasets;
				var ctx = chart.chart.ctx;
        datasets.forEach(function (element, index) {
				var arc = chart.getDatasetMeta(index).data[0];
				var startAngle = Math.PI / 2 - arc._view.startAngle;
				var endAngle = Math.PI / 2 - arc._view.endAngle;
        var radius = (arc._view.outerRadius + arc._view.innerRadius) / 2;
        var thickness = (arc._view.outerRadius - arc._view.innerRadius) / 2 -1;

				ctx.save();
				ctx.translate(arc._view.x, arc._view.y);
				ctx.fillStyle = arc._view.backgroundColor;
				ctx.beginPath();
				ctx.arc(radius * Math.sin(startAngle), radius * Math.cos(startAngle), thickness, 0, 2 * Math.PI);
				ctx.arc(radius * Math.sin(endAngle), radius * Math.cos(endAngle), thickness, 0, 2 * Math.PI);
				ctx.closePath();
				ctx.fill();
				ctx.restore();
        })
			}
		},
	});

	// write text plugin
	Chart.pluginService.register({
		afterUpdate: function (chart) {
			if (chart.config.options.elements.center) {
				var helpers = Chart.helpers;
				var centerConfig = chart.config.options.elements.center;
				var globalConfig = Chart.defaults.global;
				var ctx = chart.chart.ctx;
        
				var fontStyle = helpers.getValueOrDefault(centerConfig.fontStyle, globalConfig.defaultFontStyle);
				var fontFamily = helpers.getValueOrDefault(centerConfig.fontFamily, globalConfig.defaultFontFamily);

				if (centerConfig.fontSize)
					var fontSize = centerConfig.fontSize;
					// figure out the best font size, if one is not specified
				else {
					ctx.save();
					var fontSize = helpers.getValueOrDefault(centerConfig.minFontSize, 1);
					var maxFontSize = helpers.getValueOrDefault(centerConfig.maxFontSize, 256);
					var maxText = helpers.getValueOrDefault(centerConfig.maxText, centerConfig.text);

					do {
						ctx.font = helpers.fontString(fontSize, fontStyle, fontFamily);
						var textWidth = ctx.measureText(maxText).width;

						// check if it fits, is within configured limits and that we are not simply toggling back and forth
						if (textWidth < chart.innerRadius * 2 && fontSize < maxFontSize)
							fontSize += 1;
						else {
							// reverse last step
							fontSize -= 1;
							break;
						}
					} while (true)
					ctx.restore();
				}

				// save properties
				chart.center = {
					font: helpers.fontString(fontSize, fontStyle, fontFamily),
					fillStyle: helpers.getValueOrDefault(centerConfig.fontColor, globalConfig.defaultFontColor)
				};
			}
		},
		afterDraw: function (chart) {
			if (chart.center) {
				var ctx = chart.chart.ctx;
        
        var i,a,s;
        var n = chart;
        var total = 0;
        for(i=0,a=(n.data.datasets||[]).length;a>i;++i){
        	s = n.getDatasetMeta(i);
          var x;
					for(x=0; x<s.data.length; x++){
        		if (!s.data[x].hidden)
            	total += n.data.datasets[i].data[x];
          }
        }
        
				ctx.save();
				ctx.font = chart.center.font;
				ctx.fillStyle = chart.center.fillStyle;
				ctx.textAlign = 'center';
				ctx.textBaseline = 'middle';
				var centerX = (chart.chartArea.left + chart.chartArea.right) / 2;
				var centerY = (chart.chartArea.top + chart.chartArea.bottom) / 2;
				ctx.fillText(total, centerX, centerY);
				ctx.restore();
			}
		},
	})


	var config = {
		type: 'doughnut',
		data: {
			labels: [
				"Red",
				"Gray"
			],
			datasets: [{
				data: [67, 33],
				backgroundColor: [
					"#FF6684",
					"#ccc"
				],
				hoverBackgroundColor: [
					"#FF6384",
					"#ccc"
				]
			},
      {
				data: [67, 33],
				backgroundColor: [
					"#FF6684",
					"#ccc"
				],
				hoverBackgroundColor: [
					"#FF6384",
					"#ccc"
				]
			}]
		},
		options: {
			elements: {
				arc: {
					roundedCornersFor: 0
				},
				center: {
					// the longest text that could appear in the center
					maxText: '100%',
					//text: '67%',
					fontColor: '#FF6684',
					fontFamily: "'Helvetica Neue', 'Helvetica', 'Arial', sans-serif",
					fontStyle: 'normal',
					// fontSize: 12,
					// if a fontSize is NOT specified, we will scale (within the below limits) maxText to take up the maximum space in the center
					// if these are not specified either, we default to 1 and 256
					minFontSize: 1,
					maxFontSize: 256,
				}
			}
		}
	};


		var ctx = document.getElementById("myChart").getContext("2d");
		var myChart = new Chart(ctx, config);
