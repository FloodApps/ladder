<template>
        <div class="ladder-menu">
            <template v-if="selected.type !='network'">
                <button @click="putSymbol('top', 'connect')" :disabled="!canPutTop" class="arrow-top"><span class="iconl-arrow-top"></span></button>
                <button @click="putSymbol('bottom', 'connect')" :disabled="!canPut"><span class="iconl-arrow-bottom"></span></button>
                <button @click="putSymbol('line', 'connect')" :disabled="!canPut" class="line"><span class="iconl-arrow-rigth"></span></button>
                <div class="contact">
                    <button @click="openMenu('contact')" :disabled="!canPut"><span class="iconl-contact"></span></button>
                    <ul v-show="menu.contact" class="submenu">
                        <li @click="putSymbol('cno','contact')">-|  &nbsp;&nbsp;|-</li>
                        <li @click="putSymbol('cnc','contact')">-| / |-</li>
                        <li @click="putSymbol('n','contact')">-| N |-</li>
                        <li @click="putSymbol('p','contact')">-| P |-</li>
                    </ul>
                </div>
                <div class="coil">
                <button @click="openMenu('coil')" :disabled="!canPutFinal"><span class="iconl-coil" ></span></button>
                    <ul v-show="menu.coil" class="submenu">
                        <li @click="putSymbol('coil','coil')">-( &nbsp;&nbsp;)</li>
                        <li @click="putSymbol('reset','coil')">R</li>
                        <li @click="putSymbol('set','coil')">S</li>
                    </ul>
                </div>
                <div class="blocks">
                    <button @click="openMenu('blocks')" :disabled="!canPutFinal"><span class="iconl-blocks"></span></button>
                    <ul v-show="menu.blocks" class="submenu">
                        <li @click="putSymbol('ton', 'block')">TON</li>
                        <li @click="putSymbol('ctu', 'block')">CTU</li>
                    </ul>
                </div>
            </template>
            <template v-if="selected.type =='network'">
                <button @click="copyNetwork()"><span class="material-icons">content_copy</span></button>
                <button @click="pasteNetwork()" :disabled="copyIndex == null"><span class="material-icons">content_paste</span></button>
            </template>
            <button @click="deleteSymbol" :disabled="!canDelete"><span class="material-icons trash">delete</span></button>
        </div>
        <div v-show="backMenu" class="backmenu" @click="closeMenu"></div>
</template>
<script>
import { computed, reactive, ref } from '@vue/runtime-core'
import {useStore} from "vuex"
export default {
    setup() {
        const store = useStore()
        const menu = reactive({
            contact: false,
            coil: false,
            blocks: false,
            principal: false,
            alert: false,
        })
        const selected = computed(() => store.state.selected)

        var copyIndex = ref(null);

        const copyNetwork = () => {
            copyIndex.value = selected.value.n
        }

        const pasteNetwork = () => {
            store.commit("addAuxRow")
            store.commit("insertNetwork", {copyIndex: copyIndex.value, insertIndex: selected.value.n-1})
            store.dispatch("saveInLocal")
            copyIndex.value = null
        }

        const openMenu = (option) => {
            const value = menu[option]
            closeMenu()
            menu[option] = !value
        }
        
        const backMenu = computed(() => {
            return menu.contact || menu.coil || menu.blocks
        })

        const closeMenu = () => {
            menu.contact = menu.coil = menu.blocks = false;
        }
        
        const putSymbol = (symbol, type) => {
            closeMenu() 
            store.dispatch("putSymbol", {symbol, type})
        }
        const deleteSymbol = () => {
            if(selected.value.type == "network")
                store.commit("deleteNetwork")
            else
                store.dispatch("deleteSymbol")
        }
        

        const canPut = computed(() => {
            if(store.getters.box({property: "symbol"}) == "center-input" || selected.value.type == 'network' || store.state.run.run)
                return false
            return store.getters.selectedFinal !== undefined ? (selected.value.b <= store.getters.selectedFinal) : true
        })
        const canPutFinal = computed(() => {
            if(!canPut.value) return false
            return selected.value.type != 'network' && selected.value.b > 0 ? (store.getters.selectedLast ? (selected.value.b >= store.getters.selectedLast) : true) : false
        })
        const canPutTop = computed(() => {
            if(!canPut.value) return false

            var r = selected.value.r
            if(selected.value.type == 'aux') 
                r = store.state.network[selected.value.n].row.length
            if(r > 0){
                const final = store.getters.row({property: "final", n: selected.value.n, r: r-1})
                return final ? (selected.value.b < final) : true 
            }
            else
                return false
        })
        const canDelete = computed(() => {
            if(store.state.run.run) return false
            if(selected.value.type == 'network') return true
            if(selected.value.type == 'aux') return false

            const symbol = store.getters.box({property: "symbol"})
            const symbolBefore = selected.value.b>0 ? store.getters.box({property: "symbol", n: selected.value.n, r: selected.value.r, b: selected.value.b-1}) : ""

            if(symbol == "continue" && selected.value.b>0 && (
                    symbolBefore == "continue" || 
                    symbolBefore == "start" || !symbolBefore ||
                    /coil|reset|set|ton-top|ctu-top|deleteInput/.test(symbolBefore) ||
                    store.getters.connection({pos: "top", n: selected.value.n, r: selected.value.r, b: selected.value.b-1}) ||
                    (selected.value.r > 0 && store.getters.connection({pos: "bottom",  n: selected.value.n, r: selected.value.r, b: selected.value.b-1}))
            )) return true
            
            return (symbol != "center-input" && symbol != "start" && symbol != "continue")
        })

        return{putSymbol, deleteSymbol, menu, closeMenu, selected, canPut, canPutFinal, canPutTop, canDelete, backMenu, openMenu, copyNetwork, pasteNetwork, copyIndex}
    },
}
</script>
<style scoped>
.panel-top {
    width: 100%;
    height: 50px;
    background-color: rgb(24, 99, 99);
    display: flex;
    align-items: center;
    justify-content: flex-start;
    padding-left: 5px;
}
.panel-top .ladder-menu{
    display: flex;
}
.panel-top button{
    border-radius: 15%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 23px;
    width: 36px;
    height: 36px;
    margin-right: 5px;
    color: rgb(41, 41, 41);
    cursor: pointer;
    border: none;
    -webkit-tap-highlight-color: rgba(0,0,0,0);
}
.panel-top button:active{
    opacity: 0.9;
}
.panel-top button:disabled{
    opacity: 0.5;
    cursor: default;
}
.menu{
    display: none;
    margin-left: auto;
    height: 100%;
}
.panel-top .menu-button{
    height: 100%;
    padding: 7px;
    background: none;
    border: none;
    margin: 0;
    -webkit-tap-highlight-color: rgba(0,0,0,0);
    user-select: none;
}
.panel-top .menu-button span{
    font-size: 30px;
    color: rgb(202, 202, 202);
}
.panel-top .menu-button:active{
    background: rgba(0, 0, 0, 0.062);
}

.symbolTable-title{
    width: 100%;
    text-align: center;
    color: rgb(181, 214, 216);
    font-size: 23px;
}

.panel-top .trash{
    color: rgb(187, 65, 65);
}

.contact, .coil, .blocks{
    position: relative;
}

.submenu{
    width: 50px; 
    position: absolute;
    top: 42px;
    left: -6px;
    background-color: rgb(24, 99, 99);
    list-style:none;
    border-left: 1px solid rgba(0, 0, 0, 0.137);
    border-right: 1px solid rgba(0, 0, 0, 0.137);
    border-top: 1px solid rgba(0, 0, 0, 0.137);
    z-index: 10;
    color: rgb(202, 219, 213);
    font-size: 18px;
    user-select: none;
}
.submenu li{
    padding-top: 10px;
    padding-bottom: 10px;
    width: 100%;
    text-align: center;
    cursor: pointer;
    border-bottom: 1px solid rgba(0, 0, 0, 0.137);
}
.submenu li:hover{
    background-color: rgb(151, 202, 182);
}

.panel-top .menu ul{
    width: 200px;
    height: calc(var(--vh) - 50px);
    right: 0;
    top: 50px;
    background: rgb(24, 99, 99);
    position: absolute;
    z-index: 10;
}

.panel-top .menu li{
    display: flex;
    align-items: center;
    justify-content: center;
    padding-top: 20px;
    padding-bottom: 20px;
    -webkit-tap-highlight-color: rgba(0,0,0,0);
    color: rgb(202, 202, 202);
    font-size: 20px;
    user-select: none;
}

.panel-top .menu li span{
    font-size: 30px;
    margin-right: 4px;
}

.backmenu{
    position: absolute;
    width: 100%;
    height: calc(var(--vh) - 50px);
    z-index: 5;
    top: 50px;
    left: 0;
}

@media screen and (max-width: 768px) {
    .menu{
        display: block;
    }
}
@media screen and (max-width: 330px) {
    .panel-top .line, .panel-top .arrow-top{
        display: none;
    }
}
</style>