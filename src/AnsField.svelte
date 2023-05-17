<script>
    import { answers } from "./store.js";
    import { isValid } from "./store.js";
    export let index;
    //variables
    let dateErr = false;
    let drErr = false;
    let crErr = false;
    let typeErr = false;
    let isInitial = true;
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
        let valid = true;
        if (isNaN(dr)){
            drErr = true;
            valid = false;
        } 
        else 
            drErr = false;

        if (isNaN(cr)){
            crErr = true;
            valid = false;
        }
        else
            crErr = false;

        if (type == "" && !isInitial)
            typeErr = true;
        if (type == "")
            valid = false;
        else
            typeErr = false;

        const regex = /^([1-9]|0[1-9]|1[0-2])\/\d{2}$/;
        if (!regex.test(date) || date == "")
            valid = false;

        //Input is valid
        isValid.update(values => {
            let newValues = [...values];
            newValues[index] = valid;
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
    isInitial = false;
</script>
<td>
    <div class="tableCell">
        <div class="warningWrapper">
            <img src="../static/warning.png" class="icon" class:hidden={!dateErr} alt="warning">
            <p class="errMsg">Date must be formated as mm/dd</p>
        </div>
        <input id="dateTxt" type="text" placeholder="mm/dd" bind:value={date} on:input={valuesChanged} on:blur={dateValidate}/>
    </div>
</td>
<td>
    <div class="tableCell">
        <div class="warningWrapper">
            <img src="../static/warning.png" class="icon" class:hidden={!typeErr} alt="warning">
            <p class="errMsg">Type must be selected</p>
        </div>
        <select id="accTypesDdl" bind:value={type} on:input={valuesChanged} on:blur={valuesChanged}>
        	<option>Cash</option>
        	<option>Revenue</option>
        	<option>Deferred</option>
        	<option>Receivable</option>
        	<option>Contra</option>
        	<option>System Credit (Liability)</option>
        </select>
    </div>
</td>
<td>
    <div class="tableCell">
        <div class="warningWrapper">
            <img src="../static/warning.png" class="icon" class:hidden={!drErr} alt="warning">
            <p class="errMsg">Dr must be a number</p>
        </div>
        <input id="drAmtTxt" type="text" bind:value={dr} on:input={valuesChanged}/>
    </div>
</td>
<td>
    <div class="tableCell">
        <div class="warningWrapper">
            <img src="../static/warning.png" class="icon" class:hidden={!crErr} alt="warning">
            <p class="errMsg">Cr must be a number</p>
        </div>
        <input id="crAmtTxt" name="crTxt" type="text" bind:value={cr} on:input={valuesChanged}/>
    </div>
</td>
