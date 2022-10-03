<script lang="ts">
    import Button, { Label, Icon } from "@smui/button";
    import DataTable, { Head, Body, Row, Cell } from "@smui/data-table";
    import Card from "@smui/card";
    import IconButton from "@smui/icon-button";

    import { onMount } from "svelte";

    const endpoint = "http://localhost:2222";
    const uploadEndpoint = `${endpoint}/upload`;

    let files;
    let filesInfo: Array<{ name: string; is_dir: boolean; size: number; mod_time: Date }> = [];
    $: currPath = "";

    onMount(async function () {
        await getFiles("");
    });

    async function getFiles(filename: string) {
        let path = `${endpoint}/${currPath}/${filename}`;
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
    }

    async function downloadDirectory(filename: string) {
        const link = document.createElement("a");
        link.href = `${endpoint}/${currPath}?download=${filename}`;
        link.download = `${filename}.zip`;
        link.click();
    }

    function submitForm() {
        const dataArray = new FormData();
        dataArray.append("uploadF", files[0]);
        fetch(`${endpoint}/upload`, {
            method: "POST",
            body: dataArray
        })
            .then((response) => {
                alert("SUCCESS POSTING");
                console.log(response.text());
            })
            .catch((error) => {
                console.log(error);
            });
    }

    let upFiles;

    $: if (upFiles) {
        console.log(upFiles);

        for (const file of upFiles) {
            console.log(`${file.name}: ${file.size} bytes`);
        }
    }

    $: fileUploadName = "";
    function openFileBrowser() {
        let fileInput = document.getElementById("my_file");
        fileInput.click();
    }

    function clickSubmit() {
        document.getElementById("my-submit").click();
    }

    function selectFile() {
        fileUploadName = files[0].name;
    }
</script>

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/svelte-material-ui@6.1.3/bare.min.css" />
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@20..48,100..700,0..1,-50..200" />

<main style="margin: auto; width: 50%;">
    <h3 style="color: black; width: max-content;">{currPath}</h3>
    <Card padded class="wrapper">
        <form on:submit|preventDefault={submitForm}>
            <div class="upload">
                <div>
                    <Button color="secondary" variant="outlined" class="browse-button" on:click={() => openFileBrowser()}>BROWSE...</Button>
                    <label for="my_file">{fileUploadName}</label>
                </div>
                <Button color="secondary" variant="raised" class="upload-button" on:click={() => clickSubmit()}>UPLOAD</Button>
            </div>
            <input id="my_file" bind:files type="file" hidden on:change={selectFile} />
            <input id="my-submit" type="submit" hidden />
        </form>
        <DataTable table$aria-label="Files list" class="datatable">
            <Head>
                <Row>
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
