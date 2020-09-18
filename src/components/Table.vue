<template>
    <div class="table-wrapper">
        <div class="label"
             v-if="tLabel"
             :style="{
                 top: (tTop - 10) + 'px',
                 left: tLeft + 'px',
                 width: (tWidth + 4) + 'px'}">
            {{ bLabel }}
        </div>
        <div class="label"
             v-if="bLabel"
             :style="{
                 top: (tTop - 10) + 'px',
                 left: tLeft + 'px',
                 width: (tWidth + 4) + 'px'}">
            {{ bLabel }}
        </div>
        <a class="table-delete"
           v-on:click="removeMyself"
           v-if="bActive"
           :style="{
               top: (tTop - 18) +'px',
               left: (tLeft + tWidth) + 'px'
               }">
            x
        </a>
        <div class="table" :style="{
                 top: tTop + 'px',
                 left: tLeft + 'px',
                 width: tWidth + 'px',
                 height: tHeight + 'px'
             }"
             v-bind:class="{'active': tActive}"
             @mousedown="selectBox"
        >
        </div>
    </div>
</template>

<script>
    export default {
        name: "Table",
        props: [
            't-top', 't-left', 't-width', 't-height', 't-label',
            'on-select', 't-active', 't-index', 'on-delete'
        ],
        methods: {
            selectBox() {
                this.onSelect(this.tIndex)
            },
            removeMyself() {
                this.onDelete(this.tIndex)
            },
        }
    }
</script>

<style lang="scss" scoped>
    .table {
        position: absolute;
        border: 2px #90ee90 solid;

        &:hover, &.active {
            background-color: rgba(144, 238, 144, .2);
        }

        z-index: 3;
    }
    .label {
        position: absolute;
        height: 10px;
        font-size: 10px;
        color: #000;
        font-weight: bold;
        background-color: #90ee90;
        z-index: 4;
    }
    .box-delete {
        position: absolute;
        z-index: 6;
        font-weight: bold;
        color: red;
        cursor: pointer;
        font-size: 24px;
        font-weight: bold;
    }
</style>