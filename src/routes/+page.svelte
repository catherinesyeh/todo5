<!-- JAVASCRIPT -->
<script>
    // import firebase and config info
    import { initializeApp, getApps, getApp } from "firebase/app";
    import { getFirestore, collection, onSnapshot, doc, updateDoc, deleteDoc, addDoc } from "firebase/firestore";
    import { firebaseConfig } from "$lib/firebaseConfig";
    import { browser } from "$app/environment";


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
</script>

<!-- HTML -->
<!-- input field + add button to enter new todo -->
<input type="text" placeholder="Add a task" bind:value={task} />
<button on:click={addTodo}>Add</button>

<!-- display todos as ordered list -->
<ol>
    {#each todos as item}
        <li class:complete={item.isComplete}>
            <span>
                <!-- todo task text -->
                {item.task}
            </span>
            <span>
                <!-- checkmark button to toggle status of todo-->
                <button on:click={() => markTodoAsComplete(item)}>✔</button>
                <button on:click={() => deleteTodo(item.id)}>✘</button>
            </span>
        </li>
        {:else}
        <!-- display message if no todos in list -->
        <p>No todos found</p>
    {/each}
    <p class="error">{error}</p>
</ol>

<svelte:window on:keydown={keyIsPressed}/>

<!-- CSS -->
<style>
    /* styling for todo list */
    .complete { /* cross out task if complete */
        text-decoration: line-through;
    }
    .error { /* error message styling */
        color: red;
    }
</style>