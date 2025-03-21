<template>
    <ul :class="level === 0 ? cx('rootList') : cx('submenu')" :tabindex="tabindex" v-bind="level === 0 ? ptm('rootList') : ptm('submenu')">
        <li v-if="submenu" :class="[cx('submenuLabel', { submenu }), getItemProp(submenu, 'class')]" :style="getItemProp(submenu, 'style')" role="presentation" v-bind="ptm('submenuLabel')">{{ getItemLabel(submenu) }}</li>
        <template v-for="(processedItem, index) of items" :key="getItemKey(processedItem)">
            <li
                v-if="isItemVisible(processedItem) && !getItemProp(processedItem, 'separator')"
                :id="getItemId(processedItem)"
                :style="getItemProp(processedItem, 'style')"
                :class="[cx('item', { processedItem }), getItemProp(processedItem, 'class')]"
                role="menuitem"
                :aria-label="getItemLabel(processedItem)"
                :aria-disabled="isItemDisabled(processedItem) || undefined"
                :aria-expanded="isItemGroup(processedItem) ? isItemActive(processedItem) : undefined"
                :aria-haspopup="isItemGroup(processedItem) && !getItemProp(processedItem, 'to') ? 'menu' : undefined"
                :aria-level="level + 1"
                :aria-setsize="getAriaSetSize()"
                :aria-posinset="getAriaPosInset(index)"
                v-bind="getPTOptions(processedItem, index, 'item')"
                :data-p-active="isItemActive(processedItem)"
                :data-p-focused="isItemFocused(processedItem)"
                :data-p-disabled="isItemDisabled(processedItem)"
            >
                <div :class="cx('itemContent')" @click="onItemClick($event, processedItem)" @mouseenter="onItemMouseEnter($event, processedItem)" v-bind="getPTOptions(processedItem, index, 'itemContent')">
                    <template v-if="!templates.item">
                        <a v-ripple :href="getItemProp(processedItem, 'url')" :class="cx('itemLink')" :target="getItemProp(processedItem, 'target')" tabindex="-1" v-bind="getPTOptions(processedItem, index, 'itemLink')">
                            <component v-if="templates.itemicon" :is="templates.itemicon" :item="processedItem.item" :class="cx('itemIcon')" />
                            <span v-else-if="getItemProp(processedItem, 'icon')" :class="[cx('itemIcon'), getItemProp(processedItem, 'icon')]" v-bind="getPTOptions(processedItem, index, 'itemIcon')" />
                            <span :class="cx('itemLabel')" v-bind="getPTOptions(processedItem, index, 'itemLabel')">
                                {{ getItemLabel(processedItem) }}
                            </span>
                            <template v-if="isItemGroup(processedItem)">
                                <component v-if="templates.submenuicon" :is="templates.submenuicon" :active="isItemActive(processedItem)" :class="cx('submenuIcon')" v-bind="getPTOptions(processedItem, index, 'submenuIcon')" />
                                <component v-else :is="horizontal || mobileActive ? 'AngleDownIcon' : 'AngleRightIcon'" :class="cx('submenuIcon')" v-bind="getPTOptions(processedItem, index, 'submenuIcon')" />
                            </template>
                        </a>
                    </template>
                    <component v-else :is="templates.item" :item="processedItem.item" :hasSubmenu="isItemGroup(processedItem)" :label="getItemLabel(processedItem)" :props="getMenuItemProps(processedItem, index)"></component>
                </div>
                <div v-if="isItemVisible(processedItem) && isItemGroup(processedItem)" :class="cx('overlay')" v-bind="ptm('overlay')">
                    <div :class="cx('grid')" v-bind="ptm('grid')">
                        <div v-for="col of processedItem.items" :key="getItemKey(col)" :class="cx('column', { processedItem })" v-bind="ptm('column')">
                            <MegaMenuSub
                                v-for="submenu of col"
                                :key="getSubListKey(submenu)"
                                :id="getSubListId(submenu)"
                                :style="sx('submenu', true, { processedItem })"
                                role="menu"
                                :menuId="menuId"
                                :focusedItemId="focusedItemId"
                                :submenu="submenu"
                                :items="submenu.items"
                                :templates="templates"
                                :level="level + 1"
                                :mobileActive="mobileActive"
                                :pt="pt"
                                :unstyled="unstyled"
                                @item-click="$emit('item-click', $event)"
                                @item-mouseenter="$emit('item-mouseenter', $event)"
                            />
                        </div>
                    </div>
                </div>
            </li>
            <li
                v-if="isItemVisible(processedItem) && getItemProp(processedItem, 'separator')"
                :id="getItemId(processedItem)"
                :class="[cx('separator'), getItemProp(processedItem, 'class')]"
                :style="getItemProp(processedItem, 'style')"
                role="separator"
                v-bind="ptm('separator')"
            ></li>
        </template>
    </ul>
</template>

<script>
import { isNotEmpty, resolve } from '@primeuix/utils/object';
import BaseComponent from '@primevue/core/basecomponent';
import AngleDownIcon from '@primevue/icons/angledown';
import AngleRightIcon from '@primevue/icons/angleright';
import Ripple from 'primevue/ripple';
import { mergeProps } from 'vue';

export default {
    name: 'MegaMenuSub',
    hostName: 'MegaMenu',
    extends: BaseComponent,
    emits: ['item-click', 'item-mouseenter'],
    props: {
        menuId: {
            type: String,
            default: null
        },
        focusedItemId: {
            type: String,
            default: null
        },
        horizontal: {
            type: Boolean,
            default: false
        },
        submenu: {
            type: Object,
            default: null
        },
        mobileActive: {
            type: Boolean,
            default: false
        },
        items: {
            type: Array,
            default: null
        },
        level: {
            type: Number,
            default: 0
        },
        templates: {
            type: Object,
            default: null
        },
        activeItem: {
            type: Object,
            default: null
        },
        tabindex: {
            type: Number,
            default: 0
        }
    },
    methods: {
        getSubListId(processedItem) {
            return `${this.getItemId(processedItem)}_list`;
        },
        getSubListKey(processedItem) {
            return this.getSubListId(processedItem);
        },
        getItemId(processedItem) {
            return `${this.menuId}_${processedItem.key}`;
        },
        getItemKey(processedItem) {
            return this.getItemId(processedItem);
        },
        getItemProp(processedItem, name, params) {
            return processedItem && processedItem.item ? resolve(processedItem.item[name], params) : undefined;
        },
        getItemLabel(processedItem) {
            return this.getItemProp(processedItem, 'label');
        },
        getPTOptions(processedItem, index, key) {
            return this.ptm(key, {
                context: {
                    item: processedItem.item,
                    index,
                    active: this.isItemActive(processedItem),
                    focused: this.isItemFocused(processedItem),
                    disabled: this.isItemDisabled(processedItem)
                }
            });
        },
        isItemActive(processedItem) {
            return isNotEmpty(this.activeItem) ? this.activeItem.key === processedItem.key : false;
        },
        isItemVisible(processedItem) {
            return this.getItemProp(processedItem, 'visible') !== false;
        },
        isItemDisabled(processedItem) {
            return this.getItemProp(processedItem, 'disabled');
        },
        isItemFocused(processedItem) {
            return this.focusedItemId === this.getItemId(processedItem);
        },
        isItemGroup(processedItem) {
            return isNotEmpty(processedItem.items);
        },
        onItemClick(event, processedItem) {
            this.getItemProp(processedItem, 'command', { originalEvent: event, item: processedItem.item });
            this.$emit('item-click', { originalEvent: event, processedItem, isFocus: true });
        },
        onItemMouseEnter(event, processedItem) {
            this.$emit('item-mouseenter', { originalEvent: event, processedItem });
        },
        getAriaSetSize() {
            return this.items.filter((processedItem) => this.isItemVisible(processedItem) && !this.getItemProp(processedItem, 'separator')).length;
        },
        getAriaPosInset(index) {
            return index - this.items.slice(0, index).filter((processedItem) => this.isItemVisible(processedItem) && this.getItemProp(processedItem, 'separator')).length + 1;
        },
        getMenuItemProps(processedItem, index) {
            return {
                action: mergeProps(
                    {
                        class: this.cx('itemLink'),
                        tabindex: -1
                    },
                    this.getPTOptions(processedItem, index, 'itemLink')
                ),
                icon: mergeProps(
                    {
                        class: [this.cx('itemIcon'), this.getItemProp(processedItem, 'icon')]
                    },
                    this.getPTOptions(processedItem, index, 'itemIcon')
                ),
                label: mergeProps(
                    {
                        class: this.cx('label')
                    },
                    this.getPTOptions(processedItem, index, 'label')
                ),
                submenuicon: mergeProps(
                    {
                        class: this.cx('submenuIcon')
                    },
                    this.getPTOptions(processedItem, index, 'submenuIcon')
                )
            };
        }
    },
    components: {
        AngleRightIcon: AngleRightIcon,
        AngleDownIcon: AngleDownIcon
    },
    directives: {
        ripple: Ripple
    }
};
</script>
