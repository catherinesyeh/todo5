<!-- JAVASCRIPT -->
<script>
    // import firebase and config info
    import { initializeApp, getApps, getApp } from "firebase/app";
    import { getFirestore, collection, onSnapshot, doc, updateDoc, deleteDoc, addDoc } from "firebase/firestore";
    import { firebaseConfig } from "$lib/firebaseConfig";
    import { browser } from "$app/environment";
    import html2canvas from 'html2canvas';

    // initialize firebase and database only if running on browser
    const firebaseApp = 
        browser && 
        (getApps().length === 0 ? initializeApp(firebaseConfig) : getApp());
    const db = browser && getFirestore();

    const colRef = browser && collection(db, "todos"); // get todo database

    let todos = []; // list of todos

    const unsubscribe = browser && onSnapshot(colRef, (querySnapshot) => {
        // load todos from database
        let fbTodos = []
        querySnapshot.forEach((doc) => {
            let todo = {...doc.data(), id: doc.id} // get data + id for each todo
            fbTodos = [todo, ...fbTodos]
        });
        todos = fbTodos; // assign our todo list to be equal to one from database
    });

    let task = ""; // task starts out empty
    let error = ""; // error message

    // add new todo to list
    const addTodo = async () => {
        // only add todo if nonempty task
        if (task !== "") {
            const docRef = await addDoc(collection(db, "todos"), {
            // add todo object into database
                task: task,
                isComplete: false,
                createdAt: new Date(),
            });
            task = ""; // clear task field
            error = ""; // reset error too
        } else {
            error = "Error: task is empty"; // display message if error
        }
    };

    // mark todo item as complete
    const markTodoAsComplete = async (item) => {
        // get item by id & toggle status on click
        await updateDoc(doc(db, "todos", item.id), {
            // update database too
            isComplete: !item.isComplete,
        });
    }

    // delete todo item
    const deleteTodo = async (id) => {
        // find item by id, then delete in database
        await deleteDoc(doc(db, "todos", id));
    }

    // add todos when enter key is pressed to
    const keyIsPressed = (event) => {
        if (event.key === "Enter") {
            addTodo();
        }
    }

    // save todo list to png
    const saveTodos = () => {
        html2canvas(document.body).then((canvas) =>{
            let a = document.createElement("a");
            a.download = "todos.png";
            a.href = canvas.toDataURL("image/png");
            a.click();
        });
    }
</script>

<!-- HTML -->
<div id="content">
    <h1>My Todos</h1>
    <!-- input field + add button to enter new todo -->
    <div id="input-container">
        <input type="text" placeholder="Add a task" bind:value={task} title="type todo"/>
        <button id="add-button" on:click={addTodo} title="add new todo">+</button>
    </div>
    
    <!-- display todos as ordered list -->
    <ul id="task-list">
        {#each todos as item}
            <li class:complete={item.isComplete}>
                <span class="task-text">
                    <!-- todo task text -->
                    {item.task}
                </span>
                <span>
                    <!-- checkmark button to toggle status of todo-->
                    <button class="complete-button" on:click={() => markTodoAsComplete(item)} title="toggle complete">✔</button>
                    <button class="delete-button" on:click={() => deleteTodo(item.id)} title="delete todo">✘</button>
                </span>
            </li>
            {:else}
            <!-- display message if no todos in list -->
            <p>No todos found</p>
        {/each}
        <button id="save-button" on:click={() => saveTodos()} title="save todos to png">Save todos list to .png file</button>
        <p class="error">{error}</p>
    </ul>
</div>

<!-- Key press listener -->
<svelte:window on:keydown={keyIsPressed}/>

<!-- CSS -->
<style>
    /* COLORS */
    :root {
        --text: rgb(22, 21, 21); 
        --blue: rgb(150, 195, 222);
        --light-blue: rgb(222, 236, 244);
        --green: rgb(144, 214, 146);
        --pink: rgb(236, 154, 154);
        --red: rgb(208, 68, 68);
        --gray: rgb(111, 108, 108);
        --light-gray: rgb(232, 228, 228);
    }

    ::-moz-selection { /* highlight color */
        background: var(--blue);
        color: white;
    }

    ::selection {
        background: var(--blue);
        color: white;
    }

    /* GENERAL */

    /* FONT STYLING */
    p, span, input, button { /* page text styling */
        font-family: Mulish;
        font-size: calc(14px + 0.2w);
        line-height: calc(20px + 0.4vw);
        font-weight: 300;
        color: var(--text);
        transition: 0.5s;
    }

    p:last-child, span {
        margin-bottom: 0;
    }

    h1 { /* headings */
        font-family: Be Vietnam Pro;
        font-weight: 800;
        line-height: calc(36px + 2vw);
        text-align: center;
        letter-spacing: 2px;
        margin-bottom: 5px;
    }

    /* MAIN CONTENT */
    #content {
        padding: calc(12px + 1.2vw) calc(20px + 2vw);
        text-align: center;
    }

    /* styling for todo list */
    #task-list {
        margin: 20px auto;
        width: fit-content;
        transition: 0.5s;
    }

    li { /* list elements */
        list-style: none;
        margin-bottom: 10px;
        text-align: right;
        width: 100%;
        transition: 0.5s;
    }

    .task-text {
        margin-right: 10px;
        transition: 0.5s;
    }

    .complete .task-text { /* cross out task if complete */
        text-decoration: line-through;
    }

    .error { /* error message styling */
        color: var(--red);
        margin-top: 20px;
        font-style: italic;
        font-size: smaller;
        letter-spacing: 0.5px;
        font-family: Be Vietnam Pro;
        font-weight: 400;
        transition: 0.5s;
    }

    #input-container { /* container for input + add button */
        display: inline-flex;
    }

    input, button { /* button and input field */
        border: 1px solid white;
        padding: 2px 10px;
        border-radius: 3px;
    }

    input:focus-visible, input:focus, input:active, 
    input:focus-within, input:focus-visible {
        /* outline color of input field when focused */
        outline-color: var(--blue);
        transition: 0.5s;
    }

    input {
        border-radius: 3px 0 0 3px;
    }

    button {
        background-color: var(--blue);
        color: white;
        border: none;
        cursor: pointer; 
        transition: 0.5s;
    }

    button:hover {
        opacity: 0.8;
    }

    /* special buttons */
    #add-button {
        border: 1px solid var(--blue);
        font-weight: 800;
        border-radius: 0 3px 3px 0;
        letter-spacing: 1px;
        font-size: larger;
        padding: 2px 8px;
    }

    .complete-button {
        background-color: var(--green);
    }

    .delete-button {
        background-color: var(--pink);
    }

    #save-button {
        margin-top: 10px;
    }

</style>