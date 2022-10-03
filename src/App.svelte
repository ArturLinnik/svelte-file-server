<script lang="ts">
    import Button, { Label, Icon } from "@smui/button";
    import DataTable, { Head, Body, Row, Cell } from "@smui/data-table";
    import Card from "@smui/card";
    import IconButton from "@smui/icon-button";

    import { onMount } from "svelte";

    const endpoint = "http://localhost:2222";
    const uploadEndpoint = `${endpoint}/upload`;

    let fileVar;
    let uploadFiles;

    let files;
    let filesInfo: Array<{ name: string; is_dir: boolean; size: number; mod_time: Date }> = [];
    $: currPath = "";

    onMount(async function () {
        await getFiles("");
    });

    async function getFiles(filename: string) {
        let path = `${endpoint}/${currPath}/${filename}`;
        // const response = await fetch(`${endpoint}/${path}`);
        const response = await fetch(path);

        if (response.headers.get("Content-Type") != "application/json") {
            window.open(path);
        } else {
            const data = await response.json();
            filesInfo = data.files;
            currPath = data.path;
            console.log(currPath);
        }
    }

    async function downloadFile(filename: string) {
        const link = document.createElement("a");
        link.href = `${endpoint}/${currPath}/${filename}`;
        link.download = `${filename}`;
        link.click();

        // let newPath = `${currPath}/${filename}`;
        // await fetch(`${endpoint}/${newPath}`)
        //     .then((resp) => resp.blob())
        //     .then((blob) => {
        //         const url = window.URL.createObjectURL(blob);
        //         // console.log(url);
        //         const a = document.createElement("a");
        //         a.style.display = "none";
        //         a.href = url;
        //         // the filename you want
        //         a.download = `${filename}`;
        //         document.body.appendChild(a);
        //         a.click();
        //         window.URL.revokeObjectURL(url);
        //     })
        //     .catch(() => alert("Error on download..."));
    }

    async function downloadDirectory(filename: string) {
        // pass parameter as: .../path?download
        // let path = `${currPath}/${filename}`;
        // console.log(path);

        // let response = await fetch(`${endpoint}/${currPath}?download=${filename}`);
        // console.log(await response.blob());
        const link = document.createElement("a");
        link.href = `${endpoint}/${currPath}?download=${filename}`;
        link.download = `${filename}.zip`;
        link.click();
        // link.dispatchEvent(new MouseEvent("click"));

        // await fetch(`${endpoint}/${currPath}?download=${filename}`)
        //     .then((resp) => resp.blob())
        //     .then((blob) => {
        //         const url = window.URL.createObjectURL(blob);
        //         console.log(url);
        //         const a = document.createElement("a");
        //         a.style.display = "none";
        //         a.href = url;
        //         a.download = `${filename}.zip`;
        //         document.body.appendChild(a);
        //         a.click();
        //         window.URL.revokeObjectURL(url);
        //     })
        //     .catch(() => alert("Error on download..."));

        // await fetch(`${endpoint}/${path}?download`);
    }

    // function openFile(path: string) {
    //     // e.stopPropagation();
    //     let newPath = `${currPath}/${path}`;
    //     // let file = `${endpoint}/${newPath}`;
    //     // window.open(`${endpoint}/${newPath}`, "_blank").focus();
    //     window.open(`${endpoint}/${newPath}`, "_blank");
    //     // return 0;
    // }

    // async function uploadFile(e: SubmitEvent, filename: string) {
    // async function uploadFile(e: SubmitEvent) {
    //     // const formData = new FormData(e.target as HTMLFormElement)
    //     // let path = `${endpoint}/upload`;
    //     // let response = await fetch(path, { method: "POST", body: JSON.stringify(`${currPath}/${filename}`) });
    //     // console.log(response.text());
    //     console.log("hello");
    // }

    function submitForm() {
        const dataArray = new FormData();
        // dataArray.append("file", fileVar);
        dataArray.append("uploadF", files[0]);
        fetch(`${endpoint}/upload`, {
            method: "POST",
            body: dataArray
        })
            .then((response) => {
                // Successfully uploaded
                console.log("SUCCESS POSTING");
                console.log(response.text());
            })
            .catch((error) => {
                // Upload failed
            });
    }

    let upFiles;

    $: if (upFiles) {
        // Note that `files` is of type `FileList`, not an Array:
        // https://developer.mozilla.org/en-US/docs/Web/API/FileList
        console.log(upFiles);

        for (const file of upFiles) {
            console.log(`${file.name}: ${file.size} bytes`);
        }
    }

    // const onFileSelected = (e) => {
    //     let image = e.target.files[0];
    //     let reader = new FileReader();
    //     reader.readAsDataURL(image);
    //     reader.onload = (e) => {
    //         avatar = e.target.result;
    //     };
    // };

    $: fileUploadName = "";
    function openFileBrowser() {
        let fileInput = document.getElementById("my_file");
        fileInput.click();
        // fileUploadName = fileInput[0].name;
        // document.getElementById("my_file").click();
    }

    function clickSubmit() {
        document.getElementById("my-submit").click();
    }

    // function getFileUploadName() {
    //     let fileInput = document.getElementById("my_file");
    //     fileUploadName = fileInput[0].name;
    //     // let filename = fileInput.files[0].name;
    // }

    function selectFile() {
        // alert("Selected file: " + this.value);
        fileUploadName = files[0].name;
    }
</script>

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/svelte-material-ui@6.1.3/bare.min.css" />
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@20..48,100..700,0..1,-50..200" />

<main style="margin: auto; width: 50%;">
    <h3 style="color: black; width: max-content;">{currPath}</h3>
    <Card padded class="wrapper">
        <!-- <form enctype="multipart/form-data" action={uploadEndpoint} method="post"> -->
        <!-- <form on:submit|preventDefault={submitForm}>
            <input type="file" bind:files={fileVar} name="file" />
            <br />
            <input type="submit" />
        </form> -->
        <form on:submit|preventDefault={submitForm}>
            <!-- <label for="avatar">Upload a picture:</label> -->
            <!-- <input accept="image/png, image/jpeg" bind:files id="avatar" name="avatar" type="file" /> -->
            <div class="upload">
                <div>
                    <Button color="secondary" variant="outlined" class="browse-button" on:click={() => openFileBrowser()}>BROWSE...</Button>
                    <label for="my_file">{fileUploadName}</label>
                </div>
                <!-- <label for="my_file">{upFiles.name}</label> -->
                <Button color="secondary" variant="raised" class="upload-button" on:click={() => clickSubmit()}>UPLOAD</Button>
            </div>
            <!-- <Button bind:files type="file">BROWSE</Button> -->
            <!-- <Button> -->
            <!-- BROWSE... -->
            <!-- <input bind:files type="file" hidden /> -->
            <!-- </Button> -->
            <input id="my_file" bind:files type="file" hidden on:change={selectFile} />
            <!-- <label for="file-upload" class="custom-file-upload"> -->
            <!-- hola -->
            <!-- <Button color="secondary" variant="outlined" class="upload">BROWSE...</Button> -->
            <!-- <input id="file-upload" bind:files type="file" /> -->
            <!-- </label> -->

            <!-- <input bind:files type="file" /> -->
            <input id="my-submit" type="submit" hidden />
        </form>
        <!-- <form on:submit|preventDefault={submitForm}>
            {#if avatar}
                <img class="avatar" src={avatar} alt="d" />
            {:else}
                <img class="avatar" src="https://cdn4.iconfinder.com/data/icons/small-n-flat/24/user-alt-512.png" alt="" />
            {/if}
            <img
                class="upload"
                src="https://static.thenounproject.com/png/625182-200.png"
                alt=""
                on:click={() => {
                    fileinput.click();
                }}
            />
            <div
                class="chan"
                on:click={() => {
                    fileinput.click();
                }}
            >
                Choose Image
            </div>
            <input style="display:none" type="file" accept=".jpg, .jpeg, .png" on:change={(e) => onFileSelected(e)} bind:this={fileinput} name="uploadF" />
            <input type="submit" />
        </form> -->
        <DataTable table$aria-label="Files list" class="datatable">
            <Head>
                <Row>
                    <!-- <Cell class="min" style="align-items: center" /> -->
                    <Cell>
                        {#if currPath != ""}
                            <Icon class="material-symbols-outlined clickable" on:click={() => getFiles("..")}>arrow_back</Icon>
                        {/if}
                    </Cell>
                    <Cell>Name</Cell>
                    <Cell>Size</Cell>
                    <Cell>Last Modified</Cell>
                    <Cell />
                </Row>
            </Head>
            <Body>
                <!-- {#if currPath != "."}
                    <Row class="clickable" on:click={() => getFiles("..")}><Cell /><Cell>../</Cell><Cell /><Cell /></Row>
                {/if} -->
                {#if filesInfo != null}
                    {#each filesInfo as file}
                        <Row class="clickable" on:click={() => getFiles(file.name)}>
                            {#if file.is_dir}
                                <Cell><Icon class="material-symbols-outlined">folder</Icon></Cell>
                                <Cell>{file.name}/</Cell>
                            {:else}
                                <Cell><Icon class="material-symbols-outlined">draft</Icon></Cell>
                                <Cell>{file.name}</Cell>
                            {/if}
                            <Cell>{(file.size / 1024).toFixed(2)} KB</Cell>
                            <Cell>{file.mod_time}</Cell>
                            <Cell>
                                {#if !file.is_dir}
                                    <IconButton
                                        class="material-symbols-outlined"
                                        on:click={(e) => {
                                            e.stopPropagation();
                                            downloadFile(file.name);
                                        }}>file_download</IconButton
                                    >
                                {:else}
                                    <IconButton
                                        class="material-symbols-outlined"
                                        on:click={(e) => {
                                            e.stopPropagation();
                                            downloadDirectory(file.name);
                                        }}>system_update_alt</IconButton
                                    >
                                {/if}
                            </Cell>
                        </Row>
                    {/each}
                {/if}
            </Body>
        </DataTable>
    </Card>
</main>

<style>
    :global(.wrapper) {
        margin-top: 10%;
        width: max-content;
        display: grid;
        gap: 15px;
    }

    :global(.upload) {
        display: flex;
        justify-content: space-between;
    }

    :global(.browse-button) {
        width: max-content;
    }

    :global(.upload label) {
        margin-left: 5px;
        font-style: italic;
    }

    :global(.upload-button) {
        width: max-content;
    }

    :global(.datatable) {
        min-width: 300px;
        max-width: 100%;
        /* box-shadow: 0 5px 10px 2px #00000030; */
    }

    :global(.min) {
        width: 1%;
    }

    :global(.clickable) {
        cursor: pointer;
    }

    :global(.material-symbols-outlined) {
        font-variation-settings: "FILL" 0, "wght" 400, "GRAD" 0, "opsz" 48;
    }

    :global(input[type="file"]) {
        display: none;
    }
</style>
