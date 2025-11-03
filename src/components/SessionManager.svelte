<script lang="ts">
    import { createEventDispatcher } from 'svelte';
    import { pushMsg, pushErrMsg } from '../api';
    import { t } from '../utils/i18n';

    export let sessions: ChatSession[] = [];
    export let currentSessionId: string = '';
    export let isOpen = false;

    const dispatch = createEventDispatcher();

    interface ChatSession {
        id: string;
        title: string;
        messages: any[];
        createdAt: number;
        updatedAt: number;
    }

    function formatDate(timestamp: number): string {
        const date = new Date(timestamp);
        const now = new Date();
        const diff = now.getTime() - date.getTime();
        const days = Math.floor(diff / (1000 * 60 * 60 * 24));

        if (days === 0) {
            return (
                t('aiSidebar.session.today') +
                ' ' +
                date.toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' })
            );
        } else if (days === 1) {
            return (
                t('aiSidebar.session.yesterday') +
                ' ' +
                date.toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' })
            );
        } else if (days < 7) {
            return `${days}${t('aiSidebar.session.daysAgo')}`;
        } else {
            return date.toLocaleDateString('zh-CN', { month: 'short', day: 'numeric' });
        }
    }

    function loadSession(sessionId: string) {
        dispatch('load', { sessionId });
        isOpen = false;
    }

    function deleteSession(sessionId: string, event: Event) {
        event.stopPropagation();
        dispatch('delete', { sessionId });
    }

    function newSession() {
        dispatch('new');
        isOpen = false;
    }

    function closeOnOutsideClick(event: MouseEvent) {
        const target = event.target as HTMLElement;
        if (!target.closest('.session-manager')) {
            isOpen = false;
        }
    }

    $: if (isOpen) {
        setTimeout(() => {
            document.addEventListener('click', closeOnOutsideClick);
        }, 0);
    } else {
        document.removeEventListener('click', closeOnOutsideClick);
    }

    // 按更新时间降序排序
    $: sortedSessions = [...sessions].sort((a, b) => b.updatedAt - a.updatedAt);
</script>

<div class="session-manager">
    <button
        class="session-manager__button b3-button b3-button--text"
        on:click|stopPropagation={() => (isOpen = !isOpen)}
        title={t('aiSidebar.session.title')}
    >
        <svg class="b3-button__icon"><use xlink:href="#iconHistory"></use></svg>
    </button>

    {#if isOpen}
        <div class="session-manager__dropdown">
            <div class="session-manager__header">
                <h4>{t('aiSidebar.session.history')}</h4>
                <button class="b3-button b3-button--primary" on:click={newSession}>
                    <svg class="b3-button__icon"><use xlink:href="#iconAdd"></use></svg>
                    {t('aiSidebar.session.new')}
                </button>
            </div>

            <div class="session-manager__list">
                {#if sortedSessions.length === 0}
                    <div class="session-manager__empty">{t('aiSidebar.session.empty')}</div>
                {:else}
                    {#each sortedSessions as session}
                        <div
                            class="session-item"
                            class:session-item--active={session.id === currentSessionId}
                            role="button"
                            tabindex="0"
                            on:click={() => loadSession(session.id)}
                            on:keydown={() => {}}
                        >
                            <div class="session-item__content">
                                <div class="session-item__title">{session.title}</div>
                                <div class="session-item__info">
                                    <span class="session-item__date">
                                        {formatDate(session.updatedAt)}
                                    </span>
                                    <span class="session-item__count">
                                        {session.messages.filter(m => m.role !== 'system').length}
                                        {t('aiSidebar.messages.messageCount')}
                                    </span>
                                </div>
                            </div>
                            <button
                                class="b3-button b3-button--text session-item__delete"
                                on:click={e => deleteSession(session.id, e)}
                                title={t('aiSidebar.session.delete')}
                            >
                                <svg class="b3-button__icon">
                                    <use xlink:href="#iconTrashcan"></use>
                                </svg>
                            </button>
                        </div>
                    {/each}
                {/if}
            </div>
        </div>
    {/if}
</div>

<style lang="scss">
    .session-manager {
        position: relative;
    }

    .session-manager__button {
        min-width: 32px;
    }

    .session-manager__dropdown {
        position: fixed;
        top: auto;
        right: 16px;
        background: var(--b3-theme-background);
        border: 1px solid var(--b3-border-color);
        border-radius: 8px;
        box-shadow: var(--b3-dialog-shadow);
        width: 15%;
        max-width: 320px;
        max-height: 60%;
        display: flex;
        flex-direction: column;
        z-index: 1000;
    }

    .session-manager__header {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 12px 16px;
        border-bottom: 1px solid var(--b3-border-color);

        h4 {
            margin: 0;
            font-size: 14px;
            font-weight: 600;
            color: var(--b3-theme-on-background);
        }

        button {
            font-size: 12px;
            padding: 4px 12px;
        }
    }

    .session-manager__list {
        flex: 1;
        overflow-y: auto;
        padding: 8px;
    }

    .session-manager__empty {
        padding: 40px 20px;
        text-align: center;
        color: var(--b3-theme-on-surface-light);
        font-size: 13px;
    }

    .session-item {
        display: flex;
        align-items: center;
        gap: 8px;
        padding: 12px;
        margin-bottom: 4px;
        border-radius: 6px;
        cursor: pointer;
        border: 1px solid transparent;
        transition: all 0.2s;

        &:hover {
            background: var(--b3-theme-surface);
            border-color: var(--b3-border-color);

            .session-item__delete {
                opacity: 1;
            }
        }
    }

    .session-item--active {
        background: var(--b3-theme-primary-lightest);
        border-color: var(--b3-theme-primary);

        .session-item__title {
            color: var(--b3-theme-primary);
            font-weight: 600;
        }
    }

    .session-item__content {
        flex: 1;
        min-width: 0;
    }

    .session-item__title {
        font-size: 13px;
        color: var(--b3-theme-on-background);
        margin-bottom: 4px;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }

    .session-item__info {
        display: flex;
        gap: 12px;
        font-size: 11px;
        color: var(--b3-theme-on-surface-light);
    }

    .session-item__delete {
        opacity: 0;
        transition: opacity 0.2s;
        flex-shrink: 0;

        &:hover {
            color: var(--b3-theme-error);
        }
    }

    .session-item--active .session-item__delete {
        opacity: 1;
    }
</style>
