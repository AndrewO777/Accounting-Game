<script>
    //importing components
	import AnsField from "./AnsField.svelte";
    import { answers } from "./store.js";
    import { isValid } from "./store.js";
    let answersJson;
    answers.subscribe(value => { answersJson = value; });
	let index = 0;
	let type = "Cash";
    let popupText = "Sorry that's not right, try again.";
    let popupBtnText = "Close";
    let active = false;
    let isRight = true;
    let valid = false;
    let errMsg = false;
	async function getQuestions() {
        try {
            //getting question from json
		    let res = await fetch("https://reclique.github.io/web-dev-testing/1_accounting_game/questions.json");
            if (res.ok){
                return res.json();
            } else {
                throw new Error("Could not load questions");
            }
        } catch (err) {
            throw new Error(err);
        }
	}
	function nextQuestion() {
        //add logic for checking if question was correct
        isValid.subscribe(values => {
            if (values.length == ((type == "Cash") ? result[index].correct_answers[0].entries.length : result[index].correct_answers[1].entries.length))
                valid = true;
            values.forEach(value => {
                if (!value)
                    valid = false;
            });
        });
        if (!valid){
            errMsg = true;
            return;
        } else { errMsg = false;}
        isRight = true;
        let i = (type == "Cash") ? 0 : 1;
        result[index].correct_answers[i].entries.forEach((element,i) => {
        //Json fields are when type dr and cr
        if (answersJson[i].date != element.when || answersJson[i].type.toLowerCase() != element.type){
            isRight = false;
                return;
            }
            else if (element.hasOwnProperty("Dr") && answersJson[i].dr != element.Dr){
                isRight = false;
                return;
            }
            else if (element.hasOwnProperty("Cr") && answersJson[i].cr != element.Cr){
                isRight = false;
                return;
            }
            else if (!element.hasOwnProperty("Dr") && answersJson[i].dr != 0){
                isRight = false;
                return;
            }
            else if (!element.hasOwnProperty("Cr") && answersJson[i].cr != 0){
                isRight = false;
                return;
            }
        });
        if (isRight)
        {
            popupText = "That's right!";
            popupBtnText = "Next";
        }
        active = true;
	}
    function closePopup() {
        if (isRight)
        {
			if (index < Object.keys(result).length - 1) {
				index++;
			} else {
				index = 0;
                popupText = "Congrats you got them all right!";
                popupBtnText = "Restart";
                isRight = false;
                return;
			}
        }
        active = false;
        valid = false;
        popupText = "Sorry that's not right, try again.";
        popupBtnText = "Close";
    }
	const promise = getQuestions();
	let result;
    promise.then(
        (value) => { result = value; },
        (error) => { result = error; }
    );
</script>
{#await promise}
    <!--still waiting on fetching questions-->
	<p>...waiting</p>
{:then results}
    <div class="overlay" class:hidden={!active}>
        <div class="popup">
            <h2>{popupText}</h2>
            <button on:click|preventDefault={closePopup}>{popupBtnText}</button>
        </div>
    </div>
	<h1>Accounting Questions</h1>
	<p>{results[index].title}</p>
	<p>{results[index].description}?</p>
	<form id="questionsForm" name="questionsForm">
        <div class="typeDiv">
		<label for="typeDdl" style="margin-bottom: 1em;">Type</label>
		<select bind:value={type} id="typeDdl" name="typeDdl" on:change={()=>{
                answers.set([]);
                isValid.set([]);
            }}>
			<option>Cash</option>
			<option>Accrual</option>
		</select>
        </div>
        <table>
            <tr>
                <th>Date</th>
                <th>Type</th>
                <th>Dr</th>
                <th>Cr</th>
            </tr>
		{#if type.toLowerCase() == result[index].correct_answers[0].type}
		{#each results[index].correct_answers[0].entries as question, i}
            <tr>
			    <AnsField index={i}/>
            </tr>
		{/each}
		{:else}
		{#each results[index].correct_answers[1].entries as question, i}
            <tr>
			    <AnsField index={i}/>
            </tr>
		{/each}
		{/if}
        </table>
        <p class="errMsg" class:hidden={!errMsg}>Please make sure the information in the above fields is valid, date cannot be blank</p>
		<button on:click|preventDefault={nextQuestion}>Submit</button>
	</form>
{:catch error}
	<p style="color:red">{error.message}</p>
{/await}
