<script lang="ts">
    import { createEventDispatcher, onDestroy } from 'svelte';
    import type { ProviderConfig, CustomProviderConfig } from '../defaultSettings';
    import { t } from '../utils/i18n';

    export let providers: Record<string, any>;
    export let selectedModels: Array<{ provider: string; modelId: string }> = [];
    export let isOpen = false;
    export let enableMultiModel = false; // 是否启用多模型模式

    const dispatch = createEventDispatcher();

    interface ProviderInfo {
        id: string;
        name: string;
        config: ProviderConfig;
    }

    const builtInProviderNames: Record<string, string> = {
        gemini: t('platform.builtIn.gemini'),
        deepseek: t('platform.builtIn.deepseek'),
        openai: t('platform.builtIn.openai'),
        volcano: t('platform.builtIn.volcano'),
        moonshot: t('platform.builtIn.moonshot'),
    };

    let expandedProviders: Set<string> = new Set();
    let selectedModelSet: Set<string> = new Set();

    // 生成模型唯一键
    function getModelKey(provider: string, modelId: string): string {
        return `${provider}:::${modelId}`;
    }

    // 解析模型键
    function parseModelKey(key: string): { provider: string; modelId: string } {
        const [provider, modelId] = key.split(':::');
        return { provider, modelId };
    }

    // 初始化已选择的模型集合
    $: {
        selectedModelSet = new Set(selectedModels.map(m => getModelKey(m.provider, m.modelId)));
    }

    function getProviderList(): ProviderInfo[] {
        const list: ProviderInfo[] = [];

        // 添加内置平台
        Object.keys(builtInProviderNames).forEach(id => {
            const config = providers[id];
            if (config && config.models && config.models.length > 0) {
                list.push({
                    id,
                    name: builtInProviderNames[id],
                    config,
                });
            }
        });

        // 添加自定义平台
        if (providers.customProviders && Array.isArray(providers.customProviders)) {
            providers.customProviders.forEach((customProvider: CustomProviderConfig) => {
                if (customProvider.models && customProvider.models.length > 0) {
                    list.push({
                        id: customProvider.id,
                        name: customProvider.name,
                        config: customProvider,
                    });
                }
            });
        }

        return list;
    }

    function toggleProvider(providerId: string) {
        if (expandedProviders.has(providerId)) {
            expandedProviders.delete(providerId);
        } else {
            expandedProviders.add(providerId);
        }
        expandedProviders = expandedProviders;
    }

    function toggleModel(provider: string, modelId: string) {
        const key = getModelKey(provider, modelId);

        if (selectedModelSet.has(key)) {
            selectedModelSet.delete(key);
        } else {
            selectedModelSet.add(key);
        }
        selectedModelSet = selectedModelSet;

        // 更新selectedModels数组
        selectedModels = Array.from(selectedModelSet).map(parseModelKey);
        dispatch('change', selectedModels);
    }

    function toggleEnableMultiModel() {
        dispatch('toggleEnable', enableMultiModel);
    }

    // 获取模型名称
    function getModelName(provider: string, modelId: string): string {
        let providerConfig: any = null;

        // 查找内置平台
        if (providers[provider] && !Array.isArray(providers[provider])) {
            providerConfig = providers[provider];
        } else if (providers.customProviders && Array.isArray(providers.customProviders)) {
            // 查找自定义平台
            providerConfig = providers.customProviders.find((p: any) => p.id === provider);
        }

        if (providerConfig && providerConfig.models) {
            const model = providerConfig.models.find((m: any) => m.id === modelId);
            return model ? model.name : modelId;
        }

        return modelId;
    }

    // 获取已选择模型的名称列表
    function getSelectedModelNames(): string {
        if (selectedModels.length === 0) return '';
        return selectedModels.map(m => getModelName(m.provider, m.modelId)).join('，');
    }

    function closeOnOutsideClick(event: MouseEvent) {
        const target = event.target as HTMLElement;
        if (!target.closest('.multi-model-selector')) {
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

    onDestroy(() => {
        document.removeEventListener('click', closeOnOutsideClick);
    });
</script>

<div class="multi-model-selector">
    <button
        class="multi-model-selector__button b3-button b3-button--text"
        class:multi-model-selector__button--active={enableMultiModel}
        on:click|stopPropagation={() => (isOpen = !isOpen)}
        title={t('multiModel.title')}
    >
        <svg class="b3-button__icon">
            <use xlink:href="#iconLayout"></use>
        </svg>
        <span class="multi-model-selector__label">
            {#if enableMultiModel && selectedModels.length > 0}
                {t('multiModel.enabled')} ({selectedModels.length})
            {:else}
                {t('multiModel.title')}
            {/if}
        </span>
    </button>

    {#if isOpen}
        <div class="multi-model-selector__dropdown">
            <div class="multi-model-selector__header">
                <div class="multi-model-selector__title">
                    {t('multiModel.selectModels')}
                </div>
                <div class="multi-model-selector__toggle" on:click|stopPropagation>
                    <label>
                        <input
                            type="checkbox"
                            class="b3-switch"
                            bind:checked={enableMultiModel}
                            on:change={toggleEnableMultiModel}
                        />
                        <span class="multi-model-selector__toggle-label">
                            {t('multiModel.enable')}
                        </span>
                    </label>
                </div>
            </div>

            <div class="multi-model-selector__count-header">
                <div class="multi-model-selector__count">
                    {#if selectedModels.length > 0}
                        {t('multiModel.selected')}: {selectedModels.length} ({getSelectedModelNames()})
                    {:else}
                        {t('multiModel.selected')}: {selectedModels.length}
                    {/if}
                </div>
            </div>

            <div class="multi-model-selector__tree">
                {#each getProviderList() as provider}
                    <div class="multi-model-selector__provider">
                        <div
                            class="multi-model-selector__provider-header"
                            role="button"
                            tabindex="0"
                            on:click={() => toggleProvider(provider.id)}
                            on:keydown={() => {}}
                        >
                            <svg
                                class="multi-model-selector__expand-icon"
                                class:multi-model-selector__expand-icon--expanded={expandedProviders.has(
                                    provider.id
                                )}
                            >
                                <use xlink:href="#iconRight"></use>
                            </svg>
                            <span>{provider.name}</span>
                            <span class="multi-model-selector__provider-count">
                                ({provider.config.models.length})
                            </span>
                        </div>
                        {#if expandedProviders.has(provider.id)}
                            <div class="multi-model-selector__models">
                                {#each provider.config.models as model}
                                    {@const modelKey = getModelKey(provider.id, model.id)}
                                    {@const isSelected = selectedModelSet.has(modelKey)}
                                    <div
                                        class="multi-model-selector__model"
                                        role="button"
                                        tabindex="0"
                                        class:multi-model-selector__model--selected={isSelected}
                                        on:click={() => toggleModel(provider.id, model.id)}
                                        on:keydown={() => {}}
                                    >
                                        <div class="multi-model-selector__checkbox">
                                            <input
                                                type="checkbox"
                                                checked={isSelected}
                                                on:click|stopPropagation={() =>
                                                    toggleModel(provider.id, model.id)}
                                            />
                                        </div>
                                        <div class="multi-model-selector__model-info">
                                            <span class="multi-model-selector__model-name">
                                                {model.name}
                                            </span>
                                            <span class="multi-model-selector__model-params">
                                                T: {model.temperature} | Max: {model.maxTokens}
                                            </span>
                                        </div>
                                    </div>
                                {/each}
                            </div>
                        {/if}
                    </div>
                {/each}
                {#if getProviderList().length === 0}
                    <div class="multi-model-selector__empty">{t('multiModel.noModels')}</div>
                {/if}
            </div>
        </div>
    {/if}
</div>

<style lang="scss">
    .multi-model-selector {
        position: relative;
    }

    .multi-model-selector__button {
        display: flex;
        align-items: center;
        gap: 4px;
        padding: 4px 8px;
        font-size: 12px;
        border-radius: 4px;
        transition: all 0.2s;

        &--active {
            background: var(--b3-theme-primary-lightest);
            color: var(--b3-theme-primary);
        }
    }

    .multi-model-selector__label {
        white-space: nowrap;
    }

    .multi-model-selector__dropdown {
        position: absolute;
        bottom: 100%;
        right: 0;
        margin-bottom: 8px;
        background: var(--b3-theme-background);
        border: 1px solid var(--b3-border-color);
        border-radius: 8px;
        box-shadow: var(--b3-dialog-shadow);
        min-width: 320px;
        max-width: 450px;
        max-height: 500px;
        overflow: hidden;
        z-index: 1000;
        display: flex;
        flex-direction: column;
    }

    .multi-model-selector__header {
        padding: 12px 16px;
        border-bottom: 1px solid var(--b3-border-color);
        display: flex;
        justify-content: space-between;
        align-items: center;
        background: var(--b3-theme-surface);
    }

    .multi-model-selector__title {
        font-weight: 600;
        font-size: 14px;
        color: var(--b3-theme-on-background);
    }

    .multi-model-selector__toggle {
        font-size: 12px;

        label {
            display: flex;
            align-items: center;
            gap: 6px;
            cursor: pointer;
            user-select: none;
        }
    }

    .multi-model-selector__toggle-label {
        font-size: 12px;
        color: var(--b3-theme-on-surface);
    }

    .multi-model-selector__count-header {
        padding: 8px 16px;
        border-bottom: 1px solid var(--b3-border-color);
    }

    .multi-model-selector__count {
        font-size: 12px;
        color: var(--b3-theme-on-surface-light);
        font-weight: 500;
    }

    .multi-model-selector__tree {
        padding: 8px;
        overflow-y: auto;
        flex: 1;
    }

    .multi-model-selector__provider {
        margin-bottom: 4px;
    }

    .multi-model-selector__provider-header {
        display: flex;
        align-items: center;
        gap: 4px;
        padding: 6px 8px;
        cursor: pointer;
        border-radius: 4px;
        font-weight: 600;
        font-size: 13px;
        color: var(--b3-theme-on-background);

        &:hover {
            background: var(--b3-theme-surface);
        }
    }

    .multi-model-selector__expand-icon {
        width: 12px;
        height: 12px;
        transition: transform 0.2s;
        flex-shrink: 0;
    }

    .multi-model-selector__expand-icon--expanded {
        transform: rotate(90deg);
    }

    .multi-model-selector__provider-count {
        margin-left: auto;
        font-size: 11px;
        color: var(--b3-theme-on-surface-light);
        font-weight: normal;
    }

    .multi-model-selector__models {
        padding-left: 20px;
        margin-top: 2px;
    }

    .multi-model-selector__model {
        display: flex;
        align-items: center;
        gap: 8px;
        padding: 8px 12px;
        cursor: pointer;
        border-radius: 4px;
        margin-bottom: 2px;
        border-left: 2px solid transparent;
        transition: all 0.2s;

        &:hover {
            background: var(--b3-theme-surface);
        }

        &--selected {
            background: var(--b3-theme-primary-lightest);
            border-left-color: var(--b3-theme-primary);
        }
    }

    .multi-model-selector__checkbox {
        flex-shrink: 0;

        input[type='checkbox'] {
            cursor: pointer;
        }
    }

    .multi-model-selector__model-info {
        flex: 1;
        display: flex;
        flex-direction: column;
    }

    .multi-model-selector__model-name {
        font-size: 13px;
        color: var(--b3-theme-on-background);
        margin-bottom: 2px;
        font-weight: 500;
    }

    .multi-model-selector__model-params {
        font-size: 11px;
        color: var(--b3-theme-on-surface-light);
    }

    .multi-model-selector__empty {
        padding: 20px;
        text-align: center;
        color: var(--b3-theme-on-surface-light);
        font-size: 13px;
    }
</style>
