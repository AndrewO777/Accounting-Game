<script>
    import { answers } from "./store.js";
    import { isValid } from "./store.js";
    export let index;
    let dateErr = false;
    let drErr = false;
    let crErr = false;
	let date = "";
    let type = "";
    let dr = 0;
    let cr = 0;
    function dateValidate(){
        const regex = /^([1-9]|0[1-9]|1[0-2])\/\d{2}$/;
        if (!regex.test(date) && date != ""){
            dateErr = true;
        }
        else 
            dateErr = false;
    }
    async function valuesChanged() {
        //validation
        if (isNaN(dr)){
            drErr = true;
            return;
        } 
        else 
            drErr = false;
        if (isNaN(cr)){
            crErr = true;
            return;
        }
        else
            crErr = false;
        const regex = /^([1-9]|0[1-9]|1[0-2])\/\d{2}$/;
        if (!regex.test(date) || date == "")
            return;
        isValid.update(values => {
            let newValues = [...values];
            newValues[index] = true;
            return newValues;
        });
        //waiting for select bind to catch up
        await new Promise(code => setTimeout(code, 10));
        answers.update(values => {
            let newValues = [...values];
            newValues[index] = 
            {
                "date": date,
                "type": type,
                "dr": Number(dr),
                "cr": Number(cr)
            };
            return newValues;
        });
    }
    valuesChanged();
</script>
<td>
    <input id="dateTxt" type="text" placeholder="mm/dd" bind:value={date} on:input={valuesChanged} on:blur={dateValidate}/>
    <p class="errMsg" class:hidden={!dateErr}>Date must be formated as mm/dd</p>
</td>
<td>
    <select id="accTypesDdl" bind:value={type} on:input={valuesChanged}>
    	<option>Cash</option>
    	<option>Revenue</option>
    	<option>Deferred</option>
    	<option>Receivable</option>
    	<option>Contra</option>
    	<option>System Credit (Liability)</option>
    </select>
</td>
<td>
    <input id="drAmtTxt" type="text" bind:value={dr} on:input={valuesChanged}/>
    <p class="errMsg" class:hidden={!drErr}>Dr must be a number</p>
</td>
<td>
    <input id="crAmtTxt" name="crTxt" type="text" bind:value={cr} on:input={valuesChanged}/>
    <p class="errMsg" class:hidden={!crErr}>Cr must be a number</p>
</td>
