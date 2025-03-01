<script>
    import 'overlayscrollbars/overlayscrollbars.css';
    import { OverlayScrollbars } from 'overlayscrollbars';
	import { afterUpdate, beforeUpdate, onDestroy, onMount } from 'svelte';
	import { page } from '$app/stores';
	import { GetContentLogs, GetStateLogs } from '$lib/services/logging-service';
    import NavBar from '$lib/common/nav-bar/NavBar.svelte';
    import NavItem from '$lib/common/nav-bar/NavItem.svelte';
    import ContentLogElement from './content-log-element.svelte';
	import ConversationStateLogElement from './conversation-state-log-element.svelte';

    const contentLogTab = 1;
    const conversationStateLogTab = 2;

    /** @type {import('$conversationTypes').ConversationContentLogModel[]} */
    export let contentLogs = [];

    /** @type {import('$conversationTypes').ConversationStateLogModel[]} */
    export let convStateLogs = [];

    /** @type {import('$conversationTypes').ConversationStateLogModel?} */
    export let lastestStateLog = null;

    /** @type {boolean} */
    export let autoScroll = false;

    /** @type {() => void} */
    export let closeWindow;

    /** @type {() => void} */
    export let cleanScreen;

    /** @type {any[]} */
    let scrollbars = [];
    /** @type {number} */
    let selectedTab = contentLogTab;
    let tabChanged = false;

    const options = {
		scrollbars: {
			visibility: 'auto',
			autoHide: 'move',
			autoHideDelay: 100,
			dragScroll: true,
			clickScroll: false,
			theme: 'os-theme-dark',
			pointers: ['mouse', 'touch', 'pen']
		}
	};

    onMount(async () => {
        const conversationId = $page.params.conversationId;
        contentLogs = await GetContentLogs(conversationId);
        convStateLogs = await GetStateLogs(conversationId);
        lastestStateLog = convStateLogs.slice(-1)[0];

        const scrollbarElements = [
            document.querySelector('.content-log-scrollbar'),
            document.querySelector('.conv-state-log-scrollbar')
        ].filter(Boolean);
        scrollbarElements.forEach(elem => {
            scrollbars = [ ...scrollbars, OverlayScrollbars(elem, options) ];
        });
		scrollToBottom();
	});

    beforeUpdate(() => {
        if (tabChanged) {
            autoScroll = false;
            tabChanged = false;
        }
    });

    afterUpdate(() => {
        refresh();
    });

    onDestroy(() => {
        cleanLogs();
    });

    function refresh() {
        if (autoScroll) {
            scrollToBottom();
        }
        
    }

    function scrollToBottom() {
        // @ts-ignore
        scrollbars.forEach(scrollbar => {
            setTimeout(() => {
                const { viewport } = scrollbar.elements();
                viewport.scrollTo({ top: viewport.scrollHeight, behavior: 'smooth' });
            }, 200);
        });
    }
    
    function cleanLogs() {
        contentLogs = [];
        convStateLogs = [];
    }

    function handleCleanScreen() {
        cleanLogs();
        cleanScreen && cleanScreen();
    }
    
    /** @param {number} selected */
    function handleTabClick(selected) {
        if (selectedTab === selected) {
            return;
        }
        tabChanged = true;
        selectedTab = selected;
    }
</script>

<div class="chat-log">
    <div class="card mb-0 log-background log-flex">
        <div class="log-close-btn padding-side log-header">
            <div>
                <button
                    type="button"
                    class="btn btn-sm btn-secondary btn-rounded chat-send waves-effect waves-light"
                    on:click={() => handleCleanScreen()}
                >
                    <i class="bx bx-trash"></i>
                </button>
            </div>
            <div>
                <button
                    type="button"
                    class="btn btn-sm btn-secondary btn-rounded chat-send waves-effect waves-light"
                    on:click={() => closeWindow()}
                >
                    <i class="mdi mdi-window-close"></i>
                </button>
            </div>
        </div>

        <div class="content-log-scrollbar log-list padding-side log-body" class:hide={selectedTab !== contentLogTab}>
            <ul>
                {#each contentLogs as log}
                    <ContentLogElement data={log} />
                {/each}
            </ul>
        </div>

        <div class="conv-state-log-scrollbar log-list padding-side log-body" class:hide={selectedTab !== conversationStateLogTab}>
            <ul>
                {#each convStateLogs as log}
                    <ConversationStateLogElement data={log} />
                {/each}
            </ul>
        </div>

        <div class="log-footer nav-group">
            <NavBar id={'persist-log-container'}>
                <NavItem
                    navBtnId={'content-log-tab'}
                    navBtnClasses={'log-footer-nav-btn'}
                    dataBsTarget={'#content-log-tab-pane'}
                    ariaControls={'content-log-tab-pane'}
                    navBtnText={'Content Log'}
                    disabled={selectedTab === contentLogTab}
                    active={selectedTab === contentLogTab}
                    onClick={() => handleTabClick(contentLogTab)}
                />
                <NavItem
                    navBtnId={'conv-state-log-tab'}
                    navBtnClasses={'log-footer-nav-btn'}
                    dataBsTarget={'#conv-state-log-tab-pane'}
                    ariaControls={'conv-state-log-tab-pane'}
                    navBtnText={'Conversation States'}
                    disabled={selectedTab === conversationStateLogTab}
                    active={selectedTab === conversationStateLogTab}
                    onClick={() => handleTabClick(conversationStateLogTab)}
                />
            </NavBar>
        </div>
    </div>
</div>