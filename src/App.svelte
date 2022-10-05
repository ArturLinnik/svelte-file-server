<script lang="ts">
    import Button, { Label, Icon } from "@smui/button";
    import DataTable, { Head, Body, Row, Cell } from "@smui/data-table";
    import Card from "@smui/card";
    import IconButton from "@smui/icon-button";

    import { onMount } from "svelte";

    const endpoint = "http://localhost:2222";

    let files;
    let filesInfo: Array<{ name: string; is_dir: boolean; size: number; mod_time: Date }> = [];
    $: currentPath = "";
    $: fileUploadName = "";

    onMount(async function () {
        await getFiles("");
    });

    async function getFiles(filename: string) {
        let path = `${endpoint}/${currentPath}/${filename}`;
        const response = await fetch(path);

        // Get directory
        if (response.headers.get("Content-Type") != "application/json") {
            window.open(path);
        }
        // Get file
        else {
            const data = await response.json();
            filesInfo = data.files;
            currentPath = data.path;
            console.log(currentPath);
        }
    }

    async function download(file: typeof filesInfo[0]) {
        const link = document.createElement("a");

        // Download file
        if (!file.is_dir) {
            link.href = `${endpoint}/${currentPath}/${file.name}`;
            link.download = `${file.name}`;
        }
        // Download directory
        else {
            link.href = `${endpoint}/${currentPath}?download=${file.name}`;
            link.download = `${file.name}.zip`;
        }

        link.click();
    }

    function submitForm() {
        const dataArray = new FormData();
        dataArray.append("path", currentPath);
        dataArray.append("upload-file", files[0]);
        fetch(`${endpoint}/upload`, {
            method: "POST",
            body: dataArray
        })
            .then((response) => {
                alert("SUCCESS POSTING");
                console.log(response.text());
                getFiles("");
            })
            .catch((error) => {
                console.log(error);
            });
    }

    function openFileBrowser() {
        let fileInput = document.getElementById("browse-file");
        fileInput.click();
    }

    function selectFile() {
        fileUploadName = files[0].name;
    }
</script>

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/svelte-material-ui@6.1.3/bare.min.css" />
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@20..48,100..700,0..1,-50..200" />

<main style="margin: auto; width: 50%;">
    <h3 style="color: black; width: max-content;">{currentPath}</h3>
    <Card padded class="wrapper">
        <form on:submit|preventDefault={submitForm}>
            <div class="upload">
                <div>
                    <Button color="secondary" variant="outlined" class="browse-button" on:click={() => openFileBrowser()}>BROWSE...</Button>
                    <label for="browse-file">{fileUploadName}</label>
                </div>
                <Button color="secondary" variant="raised" class="upload-button">UPLOAD</Button>
            </div>
            <input id="browse-file" bind:files type="file" hidden on:change={selectFile} />
            <input id="upload-file" type="submit" hidden />
        </form>
        <DataTable table$aria-label="Files list" class="datatable">
            <Head>
                <Row>
                    <Cell>
                        {#if currentPath != ""}
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
                                <IconButton
                                    class="material-symbols-outlined"
                                    on:click={(e) => {
                                        e.stopPropagation();
                                        download(file);
                                    }}
                                >
                                    {#if file.is_dir}
                                        system_update_alt
                                    {:else}
                                        file_download
                                    {/if}
                                </IconButton>
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
