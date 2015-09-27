---
layout: default
title: Countries I've Visited
---

<h1>Countries I've Visited</h1>
<p>Getting there...</p>
<p>
	<script src="http://cdn.amcharts.com/lib/3/ammap.js" type="text/javascript"></script>
	<script src="http://cdn.amcharts.com/lib/3/maps/js/worldHigh.js" type="text/javascript"></script>
	<script src="http://cdn.amcharts.com/lib/3/themes/dark.js" type="text/javascript"></script>
	<div id="mapdiv" style="width: 1000px; height: 450px;"></div>

	<script type="text/javascript">
	var map = AmCharts.makeChart("mapdiv", {
		type: "map",
		theme: "dark",
		pathToImages : "http://cdn.amcharts.com/lib/3/images/",
		panEventsEnabled : true,
		backgroundColor : "#535364",
		backgroundAlpha : 1,
		zoomControl: {
			panControlEnabled : false,
			zoomControlEnabled : false
		},
		dataProvider : 
		{
			map : "worldHigh",
			getAreasFromMap : true,
			areas :
			[
				{
					"id": "AT",
					"showAsSelected": true
				},
				{
					"id": "BE",
					"showAsSelected": true
				},
				{
					"id": "CH",
					"showAsSelected": true
				},
				{
					"id": "CY",
					"showAsSelected": true
				},
				{
					"id": "CZ",
					"showAsSelected": true
				},
				{
					"id": "DE",
					"showAsSelected": true
				},
				{
					"id": "ES",
					"showAsSelected": true
				},
				{
					"id": "FR",
					"showAsSelected": true
				},
				{
					"id": "GB",
					"showAsSelected": true
				},
				{
					"id": "GR",
					"showAsSelected": true
				},
				{
					"id": "HR",
					"showAsSelected": true
				},
				{
					"id": "HU",
					"showAsSelected": true
				},
				{
					"id": "IE",
					"showAsSelected": true
				},
				{
					"id": "IT",
					"showAsSelected": true
				},
				{
					"id": "NL",
					"showAsSelected": true
				},
				{
					"id": "NO",
					"showAsSelected": true
				},
				{
					"id": "PL",
					"showAsSelected": true
				},
				{
					"id": "SE",
					"showAsSelected": true
				},
				{
					"id": "SK",
					"showAsSelected": true
				},
				{
					"id": "TR",
					"showAsSelected": true
				},
				{
					"id": "RU",
					"showAsSelected": true
				},
				{
					"id": "VA",
					"showAsSelected": true
				},
				{
					"id": "GI",
					"showAsSelected": true
				},
				{
					"id": "US",
					"showAsSelected": true
				},
				{
					"id": "AR",
					"showAsSelected": true
				},
				{
					"id": "BO",
					"showAsSelected": true
				},
				{
					"id": "BR",
					"showAsSelected": true
				},
				{
					"id": "PE",
					"showAsSelected": true
				},
				{
					"id": "PY",
					"showAsSelected": true
				},
				{
					"id": "BW",
					"showAsSelected": true
				},
				{
					"id": "EG",
					"showAsSelected": true
				},
				{
					"id": "MA",
					"showAsSelected": true
				},
				{
					"id": "NA",
					"showAsSelected": true
				},
				{
					"id": "TZ",
					"showAsSelected": true
				},
				{
					"id": "ZM",
					"showAsSelected": true
				},
				{
					"id": "ZW",
					"showAsSelected": true
				},
				{
					"id": "AE",
					"showAsSelected": true
				},
				{
					"id": "IN",
					"showAsSelected": true
				},
				{
					"id": "MY",
					"showAsSelected": true
				},
				{
					"id": "HK",
					"showAsSelected": true
				},
				{
					"id": "AU",
					"showAsSelected": true
				},
				{
					"id": "NZ",
					"showAsSelected": true
				}
			]
		},
		areasSettings : {
			autoZoom : true,
			color : "#B4B4B7",
			colorSolid : "#84ADE9",
			selectedColor : "#84ADE9",
			outlineColor : "#666666",
			rollOverColor : "#9EC2F7",
			rollOverOutlineColor : "#000000"
		}
	});
	</script>
</p>