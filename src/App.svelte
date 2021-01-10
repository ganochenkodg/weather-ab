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
		InputGroupText,
		Toast,
		ToastBody
	} from 'sveltestrap';
	import {
		onMount
	} from 'svelte';
	var cityA = '';
	var cityB = '';
	var cityAjson = {};
	var cityBjson = {};
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
				cityAloaded = false;
				return;
			}
			if (cityBjson.errorCode != undefined) {
				messageAlert = cityBjson.message;
				cityBloaded = false;
				return;
			}
			cityAcopied = cityAjson;
			cityBcopied = cityBjson;
			messageAlert = 'Remains ' + cityAcopied.remainingCost + ' free requests.';
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
				name: cityAcopied.location.id,
				showInLegend: true,
				toolTipContent: "{x} </br> {name} </br> <strong>Temperature: </strong> </br> Min: {y[0]} °C, Max: {y[1]} °C",
				dataPoints: []
			},
			{
				type: "rangeSplineArea",
				xValueFormatString: "YYYY-MM-DD",
				name: cityBcopied.location.id,
				showInLegend: true,
				toolTipContent: "</br> {name} </br> <strong>Temperature: </strong> </br> Min: {y[0]} °C, Max: {y[1]} °C",
				dataPoints: []
			}
		];
		cityAcopied.location.values.forEach(function(entry) {
			data[0].dataPoints.push({
				x: new Date(entry.datetimeStr),
				y: [(entry.mint - 0), (entry.maxt - 0)]
			});
		});
		cityBcopied.location.values.forEach(function(entry) {
			data[1].dataPoints.push({
				x: new Date(entry.datetimeStr),
				y: [(entry.mint - 0), (entry.maxt - 0)]
			});
		});
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

	:global(body) {
		background-color: #ffffff;
		background-color: #ffffff;
		background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 56 28' width='56' height='28'%3E%3Cpath fill='%23ebebeb' fill-opacity='0.44' d='M56 26v2h-7.75c2.3-1.27 4.94-2 7.75-2zm-26 2a2 2 0 1 0-4 0h-4.09A25.98 25.98 0 0 0 0 16v-2c.67 0 1.34.02 2 .07V14a2 2 0 0 0-2-2v-2a4 4 0 0 1 3.98 3.6 28.09 28.09 0 0 1 2.8-3.86A8 8 0 0 0 0 6V4a9.99 9.99 0 0 1 8.17 4.23c.94-.95 1.96-1.83 3.03-2.63A13.98 13.98 0 0 0 0 0h7.75c2 1.1 3.73 2.63 5.1 4.45 1.12-.72 2.3-1.37 3.53-1.93A20.1 20.1 0 0 0 14.28 0h2.7c.45.56.88 1.14 1.29 1.74 1.3-.48 2.63-.87 4-1.15-.11-.2-.23-.4-.36-.59H26v.07a28.4 28.4 0 0 1 4 0V0h4.09l-.37.59c1.38.28 2.72.67 4.01 1.15.4-.6.84-1.18 1.3-1.74h2.69a20.1 20.1 0 0 0-2.1 2.52c1.23.56 2.41 1.2 3.54 1.93A16.08 16.08 0 0 1 48.25 0H56c-4.58 0-8.65 2.2-11.2 5.6 1.07.8 2.09 1.68 3.03 2.63A9.99 9.99 0 0 1 56 4v2a8 8 0 0 0-6.77 3.74c1.03 1.2 1.97 2.5 2.79 3.86A4 4 0 0 1 56 10v2a2 2 0 0 0-2 2.07 28.4 28.4 0 0 1 2-.07v2c-9.2 0-17.3 4.78-21.91 12H30zM7.75 28H0v-2c2.81 0 5.46.73 7.75 2zM56 20v2c-5.6 0-10.65 2.3-14.28 6h-2.7c4.04-4.89 10.15-8 16.98-8zm-39.03 8h-2.69C10.65 24.3 5.6 22 0 22v-2c6.83 0 12.94 3.11 16.97 8zm15.01-.4a28.09 28.09 0 0 1 2.8-3.86 8 8 0 0 0-13.55 0c1.03 1.2 1.97 2.5 2.79 3.86a4 4 0 0 1 7.96 0zm14.29-11.86c1.3-.48 2.63-.87 4-1.15a25.99 25.99 0 0 0-44.55 0c1.38.28 2.72.67 4.01 1.15a21.98 21.98 0 0 1 36.54 0zm-5.43 2.71c1.13-.72 2.3-1.37 3.54-1.93a19.98 19.98 0 0 0-32.76 0c1.23.56 2.41 1.2 3.54 1.93a15.98 15.98 0 0 1 25.68 0zm-4.67 3.78c.94-.95 1.96-1.83 3.03-2.63a13.98 13.98 0 0 0-22.4 0c1.07.8 2.09 1.68 3.03 2.63a9.99 9.99 0 0 1 16.34 0z'%3E%3C/path%3E%3C/svg%3E");
	}
</style>

<svelte:head>
	<script src={'https://maps.googleapis.com/maps/api/js?key=' + GOOGLE_API_KEY + ' &libraries=places&language=en'} on:load={setAutocomplete}></script>
	<script src="canvasjs.min.js"></script>
</svelte:head>

<div>
	<Row>
		<Col md="3" class="mb-2">
		<InputGroup size="sm">
			<InputGroupAddon addonType="prepend">
				<InputGroupText>CITY A</InputGroupText>
			</InputGroupAddon>
			<Input id="cityAinput" type="text" placeholder="NAME" />
		</InputGroup>
		</Col>
		<Col md="3" class="mb-2">
		<InputGroup size="sm">
			<InputGroupAddon addonType="prepend">
				<InputGroupText>CITY B</InputGroupText>
			</InputGroupAddon>
			<Input id="cityBinput" type="text" bind:value={cityB} placeholder="NAME" />
		</InputGroup>
		</Col>
		<Col md="2" class="mb-2">
		<InputGroup size="sm">
			<InputGroupAddon addonType="prepend">
				<InputGroupText>FROM</InputGroupText>
			</InputGroupAddon>
			<Input type="date" id="startDateInput" bind:value={startDate} />
		</InputGroup>
		</Col>
		<Col md="2" class="mb-2">
		<InputGroup size="sm">
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
		<Button size="sm" block color="primary" on:click={Compare}>Compare</Button>
		</Col>
	</Row>
	<h6>{messageAlert}</h6>
	<Table responsive bordered class="table table-sm bg-white">
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
			{#if (cityAloaded && cityBloaded)}
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
   {/if}
		</tbody>
	</Table>
	{#if (cityAloaded && cityBloaded)}
	<Toast style="max-width: 100%;" light>
      <ToastBody>
        	<div id="chartContainer" style="height: 300px; width: 99%;"></div>
      </ToastBody>
    </Toast>
   {/if}
</div>
