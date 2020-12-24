<script>
	import axios from 'axios';
	import {
		WEATHER_API_KEY,
		GOOGLE_API_KEY
	} from './env.js';
	import {
		Button,
		Col,
		Row,
		Modal,
		ModalBody,
		ModalFooter,
		ModalHeader,
		Input,
		Label,
		Table,
		InputGroup,
		InputGroupAddon,
		InputGroupText
	} from 'sveltestrap';
	import {
		onMount
	} from 'svelte';
	var cityA = '';
	var cityB = '';
	var cityAjson = {
		"location": {
			"values": []
		}
	};
	var cityBjson = {
		"location": {
			"values": []
		}
	};
	var cityAcopied = {
		"location": {
			"values": []
		}
	};
	var cityBcopied = {
		"location": {
			"values": []
		}
	};
	var resultJson = [];
	var cityAloaded = false;
	var cityBloaded = false;
	var startDate = '2020-06-01';
	var intervalDays = 3;
	var endDate = '';
	var chart = null;
	setEndDate();

	function setAutocomplete() {
		var options = {
			types: ['(cities)'],
		};
		var inputA = document.getElementById('cityAinput');
		var autocompleteA = new google.maps.places.Autocomplete(inputA, options);
		var inputB = document.getElementById('cityBinput');
		var autocompleteB = new google.maps.places.Autocomplete(inputB, options);
	}

	function setEndDate() {
		var date = new Date(startDate);
		date.setDate(date.getDate() + (intervalDays - 1));
		var dd = date.getDate();
		var mm = date.getMonth() + 1;
		var yyyy = date.getFullYear();
		if (dd < 10) {
			dd = '0' + dd;
		}
		if (mm < 10) {
			mm = '0' + mm;
		}
		endDate = yyyy + '-' + mm + '-' + dd;
	}
	var messageAlert = '';

	function waitForDate() {
		if (cityAloaded == false || cityBloaded == false) {
			setTimeout(function() {
				waitForDate()
			}, 100);
		} else {
			if (cityAjson.errorCode != undefined) {
				messageAlert = cityAjson.message;
				return;
			}
			if (cityBjson.errorCode != undefined) {
				messageAlert = cityBjson.message;
				return;
			}
			cityAcopied = cityAjson;
			cityBcopied = cityBjson;
			messageAlert = 'Remains ' + cityAcopied.remainingCost + ' free requests.';
			console.log(cityAcopied);
			console.log(cityBcopied);
			initChart();
		};
	}

	function Compare() {
		setEndDate();
		cityAloaded = false;
		cityBloaded = false;
		cityA = document.getElementById('cityAinput').value;
		cityB = document.getElementById('cityBinput').value;
		messageAlert = '';
		if (cityA == '' || cityB == '') {
			messageAlert = 'Set both cities names.';
			return;
		}
		loadData();
		waitForDate();
	}

	function loadData() {
		var uri = "https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/weatherdata/history?";
		uri += "unitGroup=metric" +
			"&aggregateHours=24" +
			"&contentType=json" +
			"&iconSet=icons1" +
			"&startDateTime=" + startDate +
			"&endDateTime=" + endDate +
			"&shortColumnNames=true" +
			"&locationMode=single" +
			"&outputDateTimeFormat=yyyy-MM-dd" +
			"&key=" + WEATHER_API_KEY +
			"&location=";
		resultJson = [];
		axios.get(uri + cityB)
			.then((res) => {
				cityBjson = res.data;
				cityBloaded = true;
			});
		axios.get(uri + cityA)
			.then((res) => {
				cityAjson = res.data;
				cityAloaded = true;
			});
	}

	function initChart() {
		CanvasJS.addColorSet("colorset",
			[
				"#FF9191",
				"#A6C9FF",
			]);
		var data = [{
				type: "rangeSplineArea",
				xValueFormatString: "YYYY-MM-DD",
				name:  cityAcopied.location.id,
				showInLegend: true,
				toolTipContent: "{x} </br> {name} </br> <strong>Temperature: </strong> </br> Min: {y[0]} °C, Max: {y[1]} °C",
				dataPoints: []
			},
			{
				type: "rangeSplineArea",
				xValueFormatString: "YYYY-MM-DD",
				name:  cityBcopied.location.id,
				showInLegend: true,
				toolTipContent: "</br> {name} </br> <strong>Temperature: </strong> </br> Min: {y[0]} °C, Max: {y[1]} °C",
				dataPoints: []
			}
		];
		cityAcopied.location.values.forEach(function(entry) {
      data[0].dataPoints.push({x: new Date(entry.datetimeStr), y: [(entry.mint-0),(entry.maxt-0)]});
    });
		cityBcopied.location.values.forEach(function(entry) {
			data[1].dataPoints.push({x: new Date(entry.datetimeStr), y: [(entry.mint-0),(entry.maxt-0)]});
		});
		console.log(data);
		chart = new CanvasJS.Chart("chartContainer", {
			theme: "light2",
			colorSet: "colorset",
			title: {
				text: "Temperatures graph",
				fontSize: 17,
			},
			axisX: {
				valueFormatString: "YYYY-MM-DD",
			},
			axisY: {
				title: "Temperature (°C)",
				suffix: " °C"
			},
			toolTip: {
				shared: true
			},
			data: data
		});
		chart.render();
	}
</script>

<style>
	img {
		height: 20px;
		width: 20px;
	}

	.cityarow {
		background-color: #FFF0F0;
	}

	.citybrow {
		background-color: #E0ECFF;
	}
</style>

<svelte:head>
	<script src={'https://maps.googleapis.com/maps/api/js?key=' + GOOGLE_API_KEY + ' &libraries=places'} on:load={setAutocomplete}></script>
	<script src="canvasjs.min.js"></script>
</svelte:head>

<div>
	<Row>
		<Col md="3" class="mb-2">
		<InputGroup>
			<InputGroupAddon addonType="prepend">
				<InputGroupText>CITY A</InputGroupText>
			</InputGroupAddon>
			<Input id="cityAinput" type="text" placeholder="NAME" />
		</InputGroup>
		</Col>
		<Col md="3" class="mb-2">
		<InputGroup>
			<InputGroupAddon addonType="prepend">
				<InputGroupText>CITY B</InputGroupText>
			</InputGroupAddon>
			<Input id="cityBinput" type="text" bind:value={cityB} placeholder="NAME" />
		</InputGroup>
		</Col>
		<Col md="2" class="mb-2">
		<InputGroup>
			<InputGroupAddon addonType="prepend">
				<InputGroupText>FROM</InputGroupText>
			</InputGroupAddon>
			<Input type="date" id="startDateInput" bind:value={startDate} />
		</InputGroup>
		</Col>
		<Col md="2" class="mb-2">
		<InputGroup>
			<InputGroupAddon addonType="prepend">
				<InputGroupText>DAYS</InputGroupText>
			</InputGroupAddon>
			<Input type="select" id="intervalDaysInput" bind:value={intervalDays}>
			<option>3</option>
			<option>7</option>
			<option>14</option>
			</Input>
		</InputGroup>
		</Col>
		<Col md="2" class="mb-2">
		<Button block color="success" on:click={Compare}>Compare</Button>
		</Col>
	</Row>
	<p></p>
	<h6>{messageAlert}</h6>
	<Table responsive bordered class="table table-sm">
		<thead>
			<tr>
				<th>Date</th>
				<th colspan="2">Name</th>
				<th colspan="2">Temp. Day °C</th>
				<th colspan="2">Temp. Night °C</th>
				<th colspan="2">Humidity %</th>
				<th colspan="2">Wind m/s</th>
				<th colspan="2">Conditions</th>
			</tr>
		</thead>
		<tbody>
			{#each cityAcopied.location.values as {datetimeStr, mint, maxt, humidity, wspd, conditions, icon}, i}
			<tr>
			<th scope="row">{ datetimeStr }</th>
			<td class="cityarow">{ cityAcopied.location.id}</td>
			<td class="citybrow">{ cityBcopied.location.id}</td>
			<td class="cityarow">{ maxt }</td>
			<td class="citybrow"> { cityBcopied.location.values[i].maxt }</td>
			<td class="cityarow">{ mint }</td>
			<td class="citybrow"> { cityBcopied.location.values[i].mint }</td>
			<td class="cityarow">{ humidity }</td>
			<td class="citybrow">{ cityBcopied.location.values[i].humidity }</td>
			<td class="cityarow">{ Math.floor (wspd / 4) }</td>
			<td class="citybrow">{ Math.floor (cityBcopied.location.values[i].wspd / 4) }</td>
			<td class="cityarow"><img src={'icons/'+icon+'.svg'}> { conditions }</td>
			<td class="citybrow"><img src={'icons/'+cityBcopied.location.values[i].icon+'.svg'}> { cityBcopied.location.values[i].conditions }</td>
			</tr>
			{/each}

		</tbody>
	</Table>
	<div id="chartContainer" style="height: 300px; width: 100%;"></div>
</div>
