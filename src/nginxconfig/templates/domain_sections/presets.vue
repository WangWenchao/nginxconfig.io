<!--
Copyright 2020 DigitalOcean

This code is licensed under the MIT License.
You may obtain a copy of the License at
https://github.com/digitalocean/nginxconfig.io/blob/master/LICENSE or https://mit-license.org/

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and / or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions :

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
-->

<template>
    <div class="container">
        <div class="header-group">
            <h3>{{ i18n.templates.domainSections.presets.presets }}</h3>
            <template v-if="$parent.$props.data.hasUserInteraction">
                <a v-if="expanded" class="button is-tiny" @click="expanded = false">
                    <i class="fas fa-angle-up"></i>
                </a>
                <a v-else class="button is-tiny" @click="expanded = true">
                    <i class="fas fa-angle-down"></i>
                </a>
            </template>
        </div>

        <template v-if="!$parent.$props.data.hasUserInteraction || expanded">
            <div v-if="$parent.$props.data.hasUserInteraction" class="message is-warning">
                <div class="message-body">
                    {{ i18n.templates.domainSections.presets.itLooksLikeYouCustomisedTheConfig }}
                </div>
            </div>

            <div class="buttons-group">
                <a v-for="(preset, key) in $props.data"
                   :class="`button${preset.computed ? ' is-primary' : ''}`"
                   @click="setPreset(key)"
                >
                    {{ preset.display }}
                </a>
            </div>
        </template>
    </div>
</template>

<script>
    import i18n from '../../i18n';
    import delegatedFromDefaults from '../../util/delegated_from_defaults';
    import computedFromDefaults from '../../util/computed_from_defaults';

    const defaults = {
        frontend: {
            default: false,
            display: i18n.templates.domainSections.presets.frontend,
            enabled: true,
            computedCheck (data) {
                return !data.php.php.computed
                    && !data.python.python.computed
                    && !data.reverseProxy.reverseProxy.computed
                    && data.routing.index.computed === 'index.html'
                    && data.routing.fallbackHtml.computed;
            },
        },
        php: {
            default: true,
            display: i18n.common.php,
            enabled: true,
            computedCheck (data) {
                return data.php.php.computed
                    && data.routing.index.computed === 'index.php'
                    && data.routing.fallbackPhp.computed
                    && !data.routing.fallbackHtml.computed
                    && !data.php.wordPressRules.computed
                    && !data.php.drupalRules.computed
                    && !data.php.magentoRules.computed;
            },
        },
        django: {
            default: false,
            display: i18n.common.django,
            enabled: true,
            computedCheck (data) {
                return data.python.python.computed
                    && data.python.djangoRules.computed
                    && !data.routing.root.computed;
            },
        },
        nodejs: {
            default: false,
            display: i18n.templates.domainSections.presets.nodeJs,
            enabled: true,
            computedCheck (data) {
                return data.reverseProxy.reverseProxy.computed
                    && !data.routing.root.computed;
            },
        },
        singlePageApplication: {
            default: false,
            display: i18n.templates.domainSections.presets.singlePageApplication,
            enabled: true,
            computedCheck (data) {
                return data.php.php.computed
                    && data.routing.index.computed === 'index.html'
                    && data.routing.fallbackHtml.computed;
            },
        },
        wordPress: {
            default: false,
            display: i18n.common.wordPress,
            enabled: true,
            computedCheck (data) {
                return data.routing.index.computed === 'index.php'
                    && data.routing.fallbackPhp.computed
                    && !data.routing.fallbackHtml.computed
                    && data.php.wordPressRules.computed
                    && !data.php.drupalRules.computed
                    && !data.php.magentoRules.computed;
            },
        },
        drupal: {
            default: false,
            display: i18n.common.drupal,
            enabled: true,
            computedCheck (data) {
                return data.routing.index.computed === 'index.php'
                    && data.routing.fallbackPhp.computed
                    && !data.routing.fallbackHtml.computed
                    && !data.php.wordPressRules.computed
                    && data.php.drupalRules.computed
                    && !data.php.magentoRules.computed;
            },
        },
        magento: {
            default: false,
            display: i18n.common.magento,
            enabled: true,
            computedCheck (data) {
                return data.routing.index.computed === 'index.php'
                    && data.routing.fallbackPhp.computed
                    && !data.routing.fallbackHtml.computed
                    && !data.php.wordPressRules.computed
                    && !data.php.drupalRules.computed
                    && data.php.magentoRules.computed;
            },
        },
    };

    export default {
        name: 'DomainPresets',                                      // Component name
        display: i18n.templates.domainSections.presets.presets,     // Display name for tab
        key: 'presets',                                             // Key for data in parent
        delegated: delegatedFromDefaults(defaults),                 // Data the parent will present here
        props: {
            data: Object,                                           // Data delegated back to us from parent
        },
        data () {
            return {
                i18n,
                expanded: false,
            };
        },
        computed: computedFromDefaults(defaults, 'presets', false), // Getters & setters for the delegated data
        watch: {
            // When any data changes, check if it still matches a preset
            '$parent.$props.data': {
                handler(data) {
                    // This might cause recursion, but seems not to
                    Object.keys(this.$props.data).forEach(preset => {
                        this.$props.data[preset].computed = this.$props.data[preset].computedCheck(data);
                    });
                },
                deep: true,
            },
        },
        methods: {
            setPreset(key) {
                // Set that we're using this preset
                Object.keys(this.$props.data).forEach(preset => this[preset] = preset === key);

                // Restore some specific defaults first
                this.$parent.resetValue('server', 'domain');
                this.$parent.resetValue('php', 'php');
                this.$parent.resetValue('php', 'wordPressRules');
                this.$parent.resetValue('php', 'drupalRules');
                this.$parent.resetValue('php', 'magentoRules');
                this.$parent.resetValue('python', 'python');
                this.$parent.resetValue('python', 'djangoRules');
                this.$parent.resetValue('reverseProxy', 'reverseProxy');
                this.$parent.resetValue('routing', 'root');
                this.$parent.resetValue('routing', 'index');
                this.$parent.resetValue('routing', 'fallbackHtml');
                this.$parent.resetValue('routing', 'fallbackPhp');

                switch (key) {
                case 'frontend':
                    this.$parent.setValue('php', 'php', false);
                    this.$parent.setValue('routing', 'index', 'index.html');
                    this.$parent.setValue('routing', 'fallbackHtml', true);
                    break;

                case 'php':
                    // Defaults should be PHP
                    break;

                case 'django':
                    this.$parent.setValue('php', 'php', false);
                    this.$parent.setValue('python', 'python', true);
                    this.$parent.setValue('python', 'djangoRules', true);
                    this.$parent.setValue('routing', 'root', false);
                    break;

                case 'nodejs':
                    this.$parent.setValue('reverseProxy', 'reverseProxy', true);
                    this.$parent.setValue('routing', 'root', false);
                    break;

                case 'singlePageApplication':
                    this.$parent.setValue('routing', 'index', 'index.html');
                    this.$parent.setValue('routing', 'fallbackHtml', true);
                    break;

                case 'wordPress':
                    this.$parent.setValue('php', 'wordPressRules', true);
                    break;

                case 'drupal':
                    this.$parent.setValue('php', 'drupalRules', true);
                    break;

                case 'magento':
                    this.$parent.setValue('php', 'magentoRules', true);
                    break;
                }
            },
        },
    };
</script>
