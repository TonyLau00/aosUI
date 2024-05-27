<div class="AOSUI h-[100%] w-[100%]">
    {#if !isWalletConnected}
    <div class="w-[100%] h-[100%] fixed top-0 left-0 flex justify-center align-center items-center z-[1000]">
        <div class="w-[100%] h-[100%] fixed top-0 left-0 bg-gray-200 opacity-70"></div>
        <div class="bg-white w-[50%] h-[300px] rounded px-2 py-4 text-left z-50">
            <div class="mb-4 border-b-2 flex">
                <button on:click={() => isCreatingWallet=false} class="w-[50%] {isCreatingWallet ? 'hover:border-red-400' : 'bg-red-400 text-white'}">Connect Wallet</button>
                <button on:click={() => isCreatingWallet=true} class="w-[50%] {isCreatingWallet ? 'bg-red-400 text-white ' : 'hover:border-red-400'}">Create Wallet</button>
            </div>
            <div>
                {#if isCreatingWallet }
                <div class="flex flex-col">
                    {#if !isLoading}
                    <!-- <button class="bg-sky-500 rounded-full text-white mr-auto ml-auto mb-4" on:click={createYouWallet}>Create Wallet</button> -->
                    <button on:click={() => window.open("https://www.arconnect.io/")} class="bg-sky-500 rounded-full text-white mr-auto ml-auto">Create in https://www.arconnect.io/</button>
                    {:else}
                    <div class="flex justify-center">
                        <button type="button" class="bg-indigo-500 ... text-white flex justify-center w-[50%]" disabled>
                            <svg class="animate-spin h-5 w-5 mr-3 ... border-2 rounded-full border-grey-200 border-l-white" viewBox="0 0 24 24">
                              <!-- ... -->
                            </svg>
                            Processing...
                          </button>
                    </div>
                    {/if}

                    {#if walletRes}
                    <div class="p-2">
                        <div>Wallet Info:</div>
                        <div>Address: {walletRes?.walletAddress}</div>
                        <div>Seed Phrase: {walletRes?.seedPhrase}</div>
                        <button class="bg-sky-500 rounded-full text-white" on:click={downloadWalletFile}>Downlaod Wallet file</button>
                    </div>
                    {/if}
                </div>
                {:else}
                <div>
                    <div class="text-center mb-8">Select Arweave Wallet</div>
                    <div class="flex space-x-4 text-white mb-4 justify-center">
                        <button on:click={doArConnectWallet}  class="bg-sky-500 rounded-full">AR Connect</button>
                        <!-- <button class="bg-sky-500 rounded-full">ARWEAVE.APP</button>
                        <button class="bg-sky-500 rounded-full">Import Wallet File</button> -->
                    </div>
                    <button on:click={doArDisconnectWallet}  class="bg-sky-500 rounded-full text-white">AR Diconnect</button>
                </div>
                {/if}
            </div>
        </div>
    </div>
    {/if}

    <div class="flex flex-col">
        <div>ctrl + v to paste</div>
        <div class="flex mb-4">
            <div placeholder="PID or Name" class="block w-[80%] text-left rounded-md border-0 py-1.5 pl-7 pr-20 text-gray-900 ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6">ProcessId: { processId || "Please connect"}</div>
            <button on:click={doAoConnect} class="bg-sky-500 text-white ml-2">Connect</button>
        </div>
        <!-- A terminal line tool  -->
        <div class="h-[500px] text-left">
            <div id="terminal" bind:this={terminal}/> 
        </div>
    </div>
    
</div>

<script lang="ts">
    import "xterm/css/xterm.css"
    import { DataItem } from "arbundles";
    import { createWallet } from "arweavekit/wallet";
    import { ArConnect } from 'arweavekit/auth';
    import {
        result,
        results,
        message,
        spawn,
        monitor,
        unmonitor,
        dryrun,
        createDataItemSigner,
    } from "@permaweb/aoconnect";
    import { onMount } from 'svelte';
    import { FitAddon } from 'xterm-addon-fit';
    import { WebLinksAddon } from 'xterm-addon-web-links';

    let walletRes: any;
    let isCreatingWallet: boolean = true;
    let isLoading: boolean = false;
    let isWalletConnected: booean = false;

    let processId: string;
    let terminal;
    let cur_line = "";
    let entries = [];
    let currPos = 0;
    let pos = 0;
    let isPasteEvent = false;
    let inUse = false;

    onMount(async () => {
        // let term = new xterm.Terminal();
        // term.open(terminal);
        // term.write('aos>');
        initTerm();
    })


    $: console.log(walletRes);

    async function checkWalletConnection() {
        try {
            const address = await ArConnect.getActiveAddress();
            if (address) {
                isWalletConnected = true;
            } else {
                isWalletConnected = false;
            }
        } catch (error) {
            console.log("Wallet is not connected!!!");
            isWalletConnected = false;
        }
    }

    async function createYouWallet() {
        console.log('create wallet');
        isLoading = true;
        walletRes = await createWallet({
            environment: 'mainnet',
            seedPhrase: true,
        });
        isLoading = false;
    }

    // download wallet.key as a json file
    function downloadWalletFile() {
        if (!walletRes) {
            alert('Please create a wallet first');
            return;
        }

        const element = document.createElement('a');
        const file = new Blob([JSON.stringify(walletRes.key)], { type: 'application/json' });
        element.href = URL.createObjectURL(file);
        element.download = 'wallet-ket.json';
        document.body.appendChild(element);
        element.click();
    }

    async function doArConnectWallet() {
        const res = await ArConnect.connect({
            permissions: ['ACCESS_ADDRESS', 'SIGN_TRANSACTION'],
        });
        checkWalletConnection();
    }

    async function doArDisconnectWallet() {
        const res = await ArConnect.disconnect();
        checkWalletConnection();
    }

    async function doAoConnect() {
        console.log('doAoConnect')
        
        const wallet = window.arweaveWallet;

        console.log("wallet", wallet);
        const signer = createDataItemSigner(wallet);
        console.log("signer", signer);
        const moduleTXID = "5l00H2S0RuPYe-V5GAI-1RgQEHFInSMr20E-3RNXJ_U";

        processId = await spawn({
            module: moduleTXID,
            scheduler: "_GQ33BkPtZrqxA84vM8Zk-N2aO0toNNu_C-l-rawrBA",
            signer: signer
        })

        console.log("processId", processId);
    }

    async function sendMessage() {
        const wallet = window.arweaveWallet;
        const signer = createDataItemSigner(wallet);

        const res = await message({
            process: "newProcess",
            signer: signer,
            
        });
        console.log(res);
    }

    async function initTerm() {
        let xterm = await import('xterm');
        let term = new xterm.Terminal({
            rendererType: 'dom',
            rows: 28,
            cols: 20,
            convertEol: false,
            disableStdin: false,
            cursorBlink: true,
            theme: {
                foreground: "#ECECEC", //字体
                background: "#000000", //背景色
                lineHeight: 20,
            },
            fontFamily: 'monospace',
        });
        term.open(terminal);
        term.prompt = () => {
            term.write('\r\n\u001b[32maos> ');
        };
        term.prompt();

        const fitAddon = new FitAddon();
        term.loadAddon(fitAddon);
        fitAddon.fit();

        term.onKey(async e => {
            const printable = !e.domEvent.altKey && !e.domEvent.altGraphKey && !e.domEvent.ctrlKey && !e.domEvent.metaKey 
            && !(e.domEvent.keyCode === 37 && term._core.buffer.x < 6);
            if (e.domEvent.keyCode === 13) {
                if (cur_line.replace(/^\s+|\s+$/g, '').length != 0) {
                    entries.push(cur_line);
                    currPos = entries.length - 1;
                    const res = await pushAOMessage(cur_line);
                    term.write('\n\x1b[2K\r\x1b[37m '.concat(res.toString()));
                    term.prompt();
                } else {
                    term.write('\n\x1b[2K\r\x1b[32maos> \x1b[37m');
                }
                cur_line = ""
            } else if (e.domEvent.keyCode === 8) {
                if (term._core.buffer.x > 5) {
                    cur_line = cur_line.slice(0, term._core.buffer.x - 6) + cur_line.slice(term._core.buffer.x - 5);
                    pos = cur_line.length - term._core.buffer.x + 6;
                    console.log('pos', pos);
                    term.write('\x1b[2K\r\u001b[32maos> \u001b[37m' + cur_line);
                    if (pos != 0) {
                        term.write('\x1b['.concat(pos.toString()).concat('D')); 
                    } else {
                        term.write('\x1b['.concat(pos.toString())); 
                    }
					if (term._core.buffer.x == 5 || term._core.buffer.x == cur_line.length + 6) {
						term.write('\x1b[1C');
					}
                }
            } else if (e.domEvent.keyCode === 38) { // Up arrow
                if (entries.length > 0) {
                    if (currPos > 0) {
                        currPos -= 1;
                    }
                    cur_line = entries[currPos];
                    term.write('\x1b[2K\r\u001b[32maos> \u001b[37m' + cur_line);
                }
            } else if (e.domEvent.keyCode === 40) { // down arrow
                currPos += 1;
                if (currPos === entries.length || entries.length === 0) {
                    currPos -= 1;
                    cur_line = "";
                    term.write('\x1b[2K\r\u001b[32maos> \u001b[37m');
                } else {
                    cur_line = entries[currPos];
                    term.write('\x1b[2K\r\u001b[32maos> \u001b[37m' + cur_line);
                }
            } else if (e.domEvent.keyCode == 86 && (e.domEvent.ctrlKey || e.domEvent.metaKey)) {
                console.log('paste')
                e.domEvent.preventDefault();
                const paste = await navigator.clipboard.readText();
                cur_line += paste;
                term.write(paste);
            } else if (printable && !(e.domEvent.keyCode == 39 && term._core.buffer.x > cur_line.length + 4)) {
                if (e.domEvent.keyCode != 37 && e.domEvent.keyCode != 39) {
                    var input = e.key;
                    if (e.domEvent.keyCode == 9) {
                        input = input = "    ";
                    }
                    pos = cur_line.length - term._core.buffer.x + 5;
					cur_line = [cur_line.slice(0, term._core.buffer.x - 5), input, cur_line.slice(term._core.buffer.x - 5)].join('');
					term.write('\x1b[2K\r\u001b[32maos> \u001b[37m' + cur_line);
                    if (pos != 0) {
                        term.write('\x1b['.concat(pos.toString()).concat('D')); 
                    } else {
                        term.write('\x1b['.concat(pos.toString()));
                    }
                    
                } else {
                    term.write(e.key)
                }
            }
        });
        
    }

    async function pushAOMessage(msg) {
        // return "adsadas";
        // fetch Target from Send({Target= , ...})
        console.log("msg", msg)
        const objStrRegex = /Send\(\{(.*)\}\)/;
        const objStrMatch = msg.match(objStrRegex);
        let objStr = "";
        let tagArr = [];
        let sendData = "";
        let targetPID = processId;

        if (objStrMatch) {
            objStr = objStrMatch[1];
            console.log("objStr", objStr);
            const objArr = objStr.split(",");
            objArr.forEach(element => {
                // trim space
                let tagName = element.split("=")[0].trim();
                let tagValue = element.split("=")[1].trim();
                tagArr.push({ name: tagName, value: tagValue });

                if (tagName == "Data") {
                    sendData = tagValue;
                }

                if (tagName == "Target") {
                    targetPID = tagValue;
                }
            });
        } else {
            tagArr = [
                { name: "Action", value: "Eval" },
                { name: "Data", value: msg }
            ]
            sendData = msg;
        }

        inUse = true;
        const res = await message({
            process: targetPID,
            tags: tagArr,
            signer: createDataItemSigner(window.arweaveWallet),
            data: sendData,
        })
        console.log(res);

        let { Messages, Spawns, Output, Error } = await result({
            message: res,
            process: processId,
        })
        console.log("Messages", Messages);
        console.log("Spawns", Spawns);
        console.log("Output", Output);
        console.log("Error", Error);
        
        return Output?.data?.output || "undefined";
    }

</script>

<style>
</style>