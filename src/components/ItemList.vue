<template>
<div class="item-list-component" :style="componentStyle" v-if="!isPreview">
    <div class="resize"></div>
</div>
<div class="item-list-component" v-else>
    <PropertyComponent v-for="child in items" class="component" :isPreview="true" :data="child.data" :templateData="templateData" @changeComponentData="changeComponentData(child, $event)"></PropertyComponent>
</div>
</template>

<script>
import PropertyComponent from '../components/Property'
import {getComponentTranslate} from '../services/utils'
export default {
    props: ['isPreview', 'parent', 'data', 'templateData', 'page', 'changeComponentData'],
    components:{
        PropertyComponent
    },
    data(){
        return {
            children: [],
        }
    },
    computed: {
        componentStyle(){
            let top = this.parent ? this.data.top - this.parent.top : this.data.top
            let left = this.parent ? this.data.left - this.parent.left : this.data.left
            return {
                top: top + 'mm',
                left: left + 'mm',
                width: this.data.width + 'mm',
                height: this.data.height + 'mm',
                transform: 'rotate(' + this.data.rotateDeg + 'deg) ' + getComponentTranslate(this.data),
                transformOrigin: '0 0',
                zIndex: this.data.zIndex
            }
        },
        items(){
            //如果预览时，渲染商品列表，返回属性列表
            this.children = []
            let top = this.data.top
            let h = this.data.height
            let number = this.data.number
            //分页商品列表
            let productList = this.templateData.productList.slice((this.page -1) * number, this.page * number)
            //渲染的行数
            number = Math.min(number, productList.length)
            if(number){
                //数据之间的间距
                let gap = h/(number+1)
                //每条数据克隆每个属性，定位属性的位置，传递productIndex来定位属性所属的product
                productList.forEach((item, i) => {
                    let items = JSON.parse(JSON.stringify(this.data.children))
                    items.forEach(child => {
                        child.data.productIndex = (this.page -1) * this.data.number + i
                        child.data.top = top + (i+1) * gap - 0.5 * child.data.height
                    })
                    this.children = this.children.concat(items)
                })    
            } else {
                //还没加载数据就渲染属性的样本值，只有一行
                this.data.children.forEach(child => {
                    let childClone = JSON.parse(JSON.stringify(child))
                    childClone.data.productIndex = 0
                    childClone.data.top = top + 0.5 * h - 0.5 * childClone.data.height
                    this.children.push(childClone)
                })
            }
            return this.children
        }
    },
}
</script>

<style lang="scss">
.item-list-component {
    background-color: rgba(10, 191, 171, .2);
    .resize {
        display: none;
        position: absolute;
        opacity: 0;
        filter: alpha(opacity=0);
        width: 20px;
        height: 20px;
        top: 100%;
        left: 100%;
        margin-left: -12px;
        margin-top: -12px;
        cursor: nwse-resize;
    }
    &.active {
        border: 1px dashed #4ec0ff;
        margin-left: -1px;
        margin-top: -1px;
        display: block;
        .resize {
            display: block;
        }
    }
}
</style>