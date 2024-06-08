<script>
	let buttonsInvisible = false
	let tempoStart = false
	let tempoSignature = false
	let instructions = "start"
	let intervalsVisible = false
	let tempo
	let length
	let dictationGenerated = false
	let generateId = "generate"
	let dictStaffVisible = false
	let answerButtonVisible = false

	//removes tonal and atonal buttons
	function removeButtons() {
		buttonsInvisible = true
	}

	//random number generator
	function getRandomInt(min, max) {
		min = Math.ceil(min)
		max = Math.floor(max)
		return Math.floor(Math.random() * (max - min + 1)) + min //The maximum is inclusive and the minimum is inclusive
	}

	//creates tempo and length fields for atonal dictator
	function tempoLength() {
		tempoStart = true
	}

	//creates tempo and meter fields for tonal dictator
	function tempoMeter() {
		tempoSignature = true
	}

	//creates checkboxes for intervals for atonal dictator
	let intervals = [
		"m2",
		"M2",
		"m3",
		"M3",
		"P4",
		"TT",
		"P5",
		"m6",
		"M6",
		"m7",
		"M7",
		"P8",
	]
	function atonalSettings() {
		instructions = "tonal"
		removeButtons()
		intervalsVisible = true
		tempoLength()
		tempoMeter()
	}

	//holds user interval selection
	let intervalPop = []
	let halfStepsPop = []
	function atonalIntervalSelector(event) {
		if (event.target.checked) {
			intervalPop.unshift(event.target.id)
			halfStepsPop.unshift(event.target.value)
		} else {
			intervalPop.splice(intervalPop.indexOf(event.target.id), 1)
			halfStepsPop.splice(halfStepsPop.indexOf(event.target.value), 1)
		}
		intervalPop = [...intervalPop]
		halfStepsPop = [...halfStepsPop]
	}

	//generates atonal exercises in numerical form
	let pitches = [
		"C4",
		"C#4",
		"D4",
		"D#4",
		"E4",
		"F4",
		"F#4",
		"G4",
		"G#4",
		"A4",
		"A#4",
		"B4",
		"C5",
		"C#5",
		"D5",
		"D#5",
		"E5",
		"F5",
		"F#5",
		"G5",
		"G#5",
		"A5",
		"A#5",
		"B5",
		"C6",
	]
	let dictPop = []
	let stepsOrderPop = []
	let dictLength
	let dictTempo
	let randPitch = pitches[getRandomInt(0, pitches.length - 1)]
	let randSteps
	let upOrDown
	let nextPitch
	function atonalDictGenerator() {
		dictTempo = parseInt(tempo)
		dictLength = parseInt(length)
		dictPop.push(parseInt(pitches.indexOf(randPitch)))
		stepsOrderPop.push(
			parseInt(halfStepsPop[getRandomInt(0, halfStepsPop.length - 1)])
		)
		for (let j = 0; j < dictLength - 1; j++) {
			randSteps = parseInt(
				halfStepsPop[getRandomInt(0, halfStepsPop.length - 1)]
			)
			stepsOrderPop.push(randSteps)
			upOrDown = parseInt(getRandomInt(0, 1))
			nextPitch = parseInt(dictPop[j])
			if (upOrDown === 0) {
				if (nextPitch - randSteps < 0) {
					nextPitch += randSteps
				} else {
					nextPitch -= randSteps
				}
			} else {
				if (nextPitch + randSteps > pitches.length - 1) {
					nextPitch -= randSteps
				} else {
					nextPitch += randSteps
				}
			}
			dictPop.push(nextPitch)
		}
	}

	//converts numbers to pitches
	function atonalDictCleaner() {
		for (let k = 0; k < dictLength; k++) {
			dictPop.splice(k, 1, pitches[dictPop[k]])
		}
	}

	//makes a rhythmic sequence
	let rhythmValues = ["2n", "4n", "8n"]
	let dictRhythmPop = []
	let preadjustedRhythms = []
	let startRhythm
	let nextRhythm
	function rhythmLogic() {
		for (let n = 0; n < dictLength; n++) {
			startRhythm = rhythmValues[getRandomInt(0, rhythmValues.length - 1)]
			if (n === 0) {
				dictRhythmPop.push([startRhythm, dictPop[0]])
				preadjustedRhythms.push(startRhythm)
			} else {
				nextRhythm =
					rhythmValues[getRandomInt(0, rhythmValues.length - 1)]
				if (dictRhythmPop[n - 1][0] === "8n") {
					nextRhythm =
						rhythmValues[getRandomInt(1, rhythmValues.length - 1)]
				}
				dictRhythmPop.push([nextRhythm, dictPop[n]])
				preadjustedRhythms.push(nextRhythm)
			}
		}
	}

	//shifts rhythms over to account for the Tone.js part having to start at 0 seconds on the transport
	function rhythmCleaner() {
		dictRhythmPop.unshift(dictRhythmPop.pop())
		for (let p = 0; p < dictLength; p++) {
			if (p === 0) {
				dictRhythmPop.splice(0, 1, [0, dictPop[p]])
			} else {
				dictRhythmPop.splice(p, 1, [
					Tone.Time(dictRhythmPop[p - 1][0]) +
						Tone.Time(dictRhythmPop[p][0]),
					dictPop[p],
				])
			}
		}
	}

	//turns the generate button into play button
	let synth
	let tick
	function generateToPlay() {
		if (length === "" || tempo === "" || isNaN(length) || isNaN(tempo)) {
			dictPop = []
			length = 0
			tempo = 0
			alert("There's a problem with the provided length or tempo values.")
		} else if (parseInt(length) > 15 || parseInt(length) <= 0) {
			dictPop = []
			length = 0
			alert(
				"An exercise this long will not be able to display the notated answer correctly. Please limit the length to a value greater than 0 and less than 15."
			)
		} else if (halfStepsPop.length === 0) {
			alert("You haven't selected any intervals.")
		} else {
			atonalDictGenerator()
			atonalDictCleaner()
			rhythmLogic()
			rhythmCleaner()
			dictationGenerated = true
			generateId = "play"
			synth = new Tone.Synth({
				oscillator: {
					type: "sine3",
				},
			}).toMaster()
			tick = new Tone.MembraneSynth().toMaster()
		}
	}

	//generate show answer button
	function showAnswer() {
		answerButtonVisible = true
	}

	//generate playback sequence
	let part
	let prevTime
	let intro
	let answerStopper = 0
	function atonalPlayback() {
		prevTime = 0
		intro = new Tone.Sequence(
			function (time, pitch) {
				tick.triggerAttackRelease(pitch, "8n", time)
			},
			["C1", "C1", "C1", "C1"],
			"4n"
		)
		part = new Tone.Part(function (time, pitch) {
			if (prevTime === dictRhythmPop.length - 1) {
				if (answerStopper === 0) {
					showAnswer()
					answerStopper += 1
				}
				Tone.Transport.cancel()
				Tone.Transport.stop()
				Tone.Transport.bpm.value = 120
			}
			synth.triggerAttackRelease(
				pitch,
				Tone.Time(preadjustedRhythms[prevTime]),
				time
			)
			prevTime++
		}, dictRhythmPop)
		intro.loop = 0
		Tone.Transport.bpm.value = dictTempo
		intro.start(0)
		part.start(Tone.Time("4n") * 4)
		Tone.Transport.start()
	}

	function dictStaff() {
		dictStaffVisible = true
		//build staff and display answer
		let vexDict = []
		let numerator = 0
		let vf = new Vex.Flow.Factory({ renderer: { elementId: "dict-staff" } })
		let score = vf.EasyScore()
		let system = vf.System()

		for (let q = 0; q < dictLength; q++) {
			if (preadjustedRhythms[q] === "2n") {
				vexDict.push(`${dictPop[q]}/h`)
				numerator += 4
			} else if (preadjustedRhythms[q] === "4n") {
				vexDict.push(`${dictPop[q]}/q`)
				numerator += 2
			} else if (preadjustedRhythms[q] === "8n") {
				vexDict.push(`${dictPop[q]}/8`)
				numerator += 1
			}
		}
		system
			.addStave({
				voices: [
					score.voice(score.notes(vexDict.join(", ")), {
						time: `${numerator}/8`,
					}),
				],
			})
			.addClef("treble")
		vf.draw()
	}
</script>

<svelte:head>
	<title>Dictation Helper</title>
	<link
		href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
		rel="stylesheet"
		integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH"
		crossorigin="anonymous" />
	<script
		src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"
		integrity="sha384-YvpcrYf0tY3lHB60NNkmXc5s9fDVZLESaAA55NDzOxhy9GkcIdslK1eN7N6jIeHz"
		crossorigin="anonymous"></script>
	<script
		src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js"
		integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1"
		crossorigin="anonymous"></script>
	<script
		src="https://cdnjs.cloudflare.com/ajax/libs/vexflow/1.2.89/vexflow-min.js"
		integrity="sha256-qKiekS+f9mx79VDRffzNmNkOW7ZWBOtyrq6RN/UrHfg="
		crossorigin="anonymous"></script>
	<script
		src="https://cdnjs.cloudflare.com/ajax/libs/tone/13.0.1/Tone.min.js"></script>
</svelte:head>
<div class="spacer-xs"></div>
<div class="bg-secondary">
	<div
		class="container min-vh-100 text-light d-flex flex-column align-items-center">
		<h1 class="display-1 text-center">Dictation Helper</h1>
		<p id="instructions" class="lead fs-4 mb-4">
			{#if instructions === "start"}
				If you're on mobile, and you're experiencing issues with
				displaying the answer, try turning your phone sideways.
			{:else}
				Select intervals, set a tempo and starting pitch, then click
				generate to play the exercise back
			{/if}
		</p>
		{#if !buttonsInvisible}
			<button
				type="button"
				name="atonal"
				id="atonal-btn"
				class="btn btn-dark btn-lg btn-block text-light w-auto"
				on:click="{atonalSettings}">Atonal</button>
		{/if}
		{#if intervalsVisible}
			<div id="dict-settings">
				{#if dictationGenerated}
					<h2 class="display-4 mb-4">
						Starting Pitch: {dictPop[0]}
						<button
							class="btn btn-warning"
							type="button"
							id="back"
							on:click="{() => location.reload()}">Back</button>
					</h2>
				{:else}
					<h2 class="display-4 mb-4 text-center">
						Intervals <button
							class="btn btn-warning"
							type="button"
							id="back"
							on:click="{() => location.reload()}">Back</button>
					</h2>
					{#each intervals as interval, i}
						<div class="d-inline">
							<input
								type="checkbox"
								value="{i + 1}"
								id="{interval}"
								class="mb-3"
								on:change="{atonalIntervalSelector}" />
							<label
								for="{interval}"
								class="align-middle fw-light">{interval}</label>
						</div>
					{/each}
				{/if}
			</div>
		{/if}
		{#if tempoStart && tempoSignature}
			<div id="tempo-start" class="mb-4">
				<div class="input-group">
					<input
						id="length"
						type="text"
						class="form-control"
						placeholder="length"
						aria-label="length"
						aria-describedby="generate"
						bind:value="{length}" />
					<input
						id="tempo"
						type="text"
						class="form-control"
						placeholder="tempo"
						aria-label="tempo"
						aria-describedby="generate"
						bind:value="{tempo}" />
					<div class="input-group-append">
						{#if generateId === "play"}
							<button
								class="btn btn-success rounded-0 rounded-end-2 border-2 border-light"
								type="button"
								id="{generateId}"
								on:click="{atonalPlayback}"
								>{generateId}</button>
						{:else}
							<button
								class="btn btn-dark rounded-0 rounded-end-2 border-2 border-light"
								type="button"
								id="{generateId}"
								on:click="{generateToPlay}"
								>{generateId}</button>
						{/if}
					</div>
				</div>
			</div>
		{/if}
		{#if answerButtonVisible}
			<div id="show-answer">
				<button
					class="btn btn-block btn-danger mb-3"
					type="button"
					id="answer-button"
					on:click|once="{dictStaff}">Show Answer</button>
			</div>
		{/if}
		<div id="dict-staff" style="background: white">
			{#if dictStaffVisible}
				<svg width="100%" height="100%" preserveAspectRatio="xMidYMid"
				></svg>
			{/if}
		</div>
		<div class="spacer-xs"></div>
	</div>
</div>
