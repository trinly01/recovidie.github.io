<svelte:head>
  <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
  <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
</svelte:head>

<Col>
<Container>
	<h1>Can you survive covid-19?</h1> <h6>Predictive Model using Neural Network By Trinmar Boado</h6>
		<br />
		<FormGroup>
			<Label>Age {ageGroup}</Label>
			<Input min="0" bind:value={age} type="number" />
		</FormGroup>
		<FormGroup>
			<Label>Sex</Label>
			<Input bind:value={sex} type="select">
				<option value=""></option>
				<option value="MALE">Male</option>
				<option value="FEMALE">Female</option>
			</Input>
		</FormGroup>
		<FormGroup>
			<Label>Date announced as positive {RangeConfToRem}</Label>
			<Input bind:value={DateRepConf} type="date" />
		</FormGroup>
		<FormGroup check>
			<Label check>
				{admitted ? 'YES' : 'NO'}
				<br />
				<Input
				bind:checked={admitted}
				type="checkbox"
				label="Admitted" />
				admitted to hospital
				<br />
				<br />
			</Label>
		</FormGroup>
		<FormGroup>
			<Label>Region ({regionText})</Label>
			<Input bind:value={region} on:input={regionOnChange} type="select" >
				<option></option>
				{#each regions as reg (reg.code)}
					<option value={reg}>{reg.name}</option>
				{/each}
			</Input>
		</FormGroup>
		<FormGroup>
			<Label>Province ({provinceText})</Label>
			<Input bind:value={province} type="select" >
				<option></option>
				{#each provinces as prov (prov.code)}
					<option value={prov}>{prov.name}</option>
				{/each}
			</Input>
		</FormGroup>
		<FormGroup>
			<Row>
				<Col />
				<Button outline size="lg" color="primary" on:click={predict} >PREDICT</Button>
				<Col />
			</Row>
		</FormGroup>
</Container>

<Modal isOpen={open} {toggle}>
	{#if result.Results.output1.value.Values[0][11]}

	<ModalHeader {toggle}>Prediction</ModalHeader>
	<ModalBody>
		<h1>{(result.Results.output1.value.Values[0][11] * 100).toFixed(2)}%</h1>
		<h3>Chance of Recovery</h3>
	</ModalBody>

	{:else}

	<ModalHeader {toggle}>Incomplete Data</ModalHeader>
	<ModalBody>
		Please complete the input fields
	</ModalBody>

	{/if}
	<ModalFooter>
		<Button color="primary" on:click={toggle}>Close</Button>
	</ModalFooter>
</Modal>

<pre>
	{JSON.stringify(result, null, 1)}
</pre>
</Col>

<script>
	import { Container, Form, Row, Col, FormGroup, Label, Input, CustomInput, Button, Modal,
    ModalBody,
    ModalFooter,
    ModalHeader } from 'sveltestrap'
	import { onMount } from 'svelte';

	let region, province, age, admitted, DateRepConf, sex, result, open=false
	let regions = []
	let provinces = []

	$: lowerAgeGroup = (parseInt(age / 10) || '')  + '' + (age % 10 > 4 ? 5 : 0)
	$: higherAgeGroup = +lowerAgeGroup + 4
	$: ageGroup = +age >= 80 ? '80+' :  lowerAgeGroup + ' to ' + higherAgeGroup
	$: RangeConfToRem = DateRepConf ? getRangeConfToRem(DateRepConf) : null
	
	$: regionText = getRegionText(region)

	const toggle = () => (open = !open);

	const getRegionText = (reg) => {
		if (!reg) return ''
		if (+reg.code === 130000000) return 'NCR'
		if (reg.name === 'MIMAROPA REGION') return 'MIMAROPA'
		if (reg.name === 'AUTONOMOUS REGION IN MUSLIM MINDANAO (ARMM)') return 'BARMM'
		if (reg.name === 'REGION XIII (Caraga)') return 'CARAGA'
		const splitRegion = reg.name.split(RegExp(' \\('))
		const splitRegion0 = splitRegion[0].split(' ')
		if (splitRegion0.length > 2) {
			return splitRegion[1].replace(')', '')
		}
		return splitRegion.join(': ').replace(')', '')
	}

	$: provinceText = getProvinceText(province)

	const getProvinceText = (prov) => {
		if (!prov) return ''
		if (province.code !== prov.code) return ''
		if (+prov.code === 130000000) return 'NCR'
		return prov.name
	}


	onMount(async () => {
		const req = await fetch('https://psgc.gitlab.io/api//regions/')
		regions = [...await req.json()]
		province = undefined
		console.log(regions)
	})

	const toTitleCase = (str) => {
        return str.replace(
            /\w\S*/g,
            function(txt) {
                return txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase();
            }
        );
    }

	const getRangeConfToRem = (d1, d2) => {
		d2 = d2 || new Date();
		d1 = new Date(d1)
		var diff = d2.getTime() - d1.getTime();
		console.log(d1)
		return Math.floor(diff / (1000 * 60 * 60 * 24));
	}
	

	const regionOnChange = (e) => {
		setTimeout(async () => {
			console.log(region)
			if (+region.code === 130000000) {
				provinces = [region]
				return
			}
			const req = await fetch('https://psgc.gitlab.io/api//provinces/'+region.code)
			provinces = [...await req.json()]
			province = undefined
			console.log(provinces)
		})
	}

	const predict = async () => {
		const req = await fetch(
			'https://vercel.trinly01.vercel.app/api/recovidie',
			{
				method: 'POST',
				// headers: {
				// 	'Content-Type': 'application/json',
				// 	Authorization: 'Bearer JKSwggwO/pOcE3oKiZp8s99oApYoCxJIrKtQXpY4SVV+HF9HHd/uwsZNG/Kt0seinOeYPvuXw1BdiozHi6xnxQ=='
				// },
				redirect: 'follow',
				body: JSON.stringify({
					"Inputs": {
					"input1": {
					"ColumnNames": [
						"CaseCode",
						"Age",
						"AgeGroup",
						"Sex",
						"DateRepConf",
						"DateDIED",
						"DateRecover",
						"RangeConfToRem",
						"RemovalType",
						"Admitted",
						"RegionRes",
						"ProvRes",
						"CityMunRes",
						"Quarantined"
					],
					"Values": [
						[
						"",
						"",
						ageGroup,
						sex,
						"",
						"",
						"",
						RangeConfToRem,
						"",
						admitted ? 'YES': 'NO',
						regionText,
						provinceText,
						"",
						""
						]
					]
					}
					},
					"GlobalParameters": {}
				})
			}
		)

		result = await req.json()
		console.log(result)

		open = true
	}
</script>