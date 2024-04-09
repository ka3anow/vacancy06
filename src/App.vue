<template>
    <div class="content" @mousewheel="scrollElement">
        <div class="left-bar"></div>
        <div class="right-bar">
            <div class="header">
                <h1>Проведение ТО и мелкий ремонт</h1>
                <div class="menu-row">
                    <ul class="main-menu">
                        <li class="menu-item" :class="{active: page == 0}" @click="changePage(0)">Общее</li>
                        <li class="menu-item" :class="{active: page == 1}" @click="changePage(1)">Товары</li>
                        <li class="menu-item" :class="{active: page == 2}" @click="changePage(2)">Доп. расходы</li>
                    </ul>
                </div>
            </div>
            <div class="page" v-show="page == 0">
                <div class="mainstyle article">
                    <h4>Общая информация</h4>
                    <p>Техническое обслуживание – это комплекс мероприятий, направленных на поддержание работоспособности и продление срока службы оборудования, машин и других технических устройств. Оно включает в себя проверку, настройку, замену изношенных деталей, чистку и смазку механизмов. Техническое обслуживание может быть плановым, когда проводится через определенные интервалы времени, и внеплановым, если оборудование требует ремонта или регулировки.</p>
                    <p>Своевременное и качественное техническое обслуживание позволяет избежать аварийных ситуаций, продлить срок службы техники и снизить затраты на ее ремонт. Кроме того, регулярное техническое обслуживание повышает безопасность эксплуатации оборудования и снижает риск возникновения производственных травм. Также техническое обслуживание является одним из основных методов контроля технического состояния оборудования и его соответствия требованиям безопасности.</p>
                </div>
            </div>
            <div class="page" v-show="page == 1">
                <div class="add-string mainstyle">
                    <button class="btn btn-primary" @click="addNewRow()"><i class="icon icon-plus"></i>Добавить строку</button>
                </div>
                <div class="table-outer mainstyle">
                    <div class="table-control">
                        <span class="save-all" @click="saveAllChanges()">Сохранить изменения</span><i class="icon icon-gear" @click="changeSettings(0)"></i>
                        <div class="settings-window" v-show="showSettings > 0" ref="modalA">
                            <ul v-show="showSettings == 1">
                                <li @click="changeSettings(2)">Отображение столбцов <i class="icon icon-right"></i></li>
                                <!-- <li @click="changeSettings(3)">Порядок столбцов <i class="icon icon-right"></i></li> -->
                            </ul>
                            <ul v-show="showSettings == 2">
                                <li @click="changeSettings(1)" class="justify-content-start"><i class="icon icon-left"></i>Отображение столбцов</li>
                                <li class="justify-content-start" v-for="(item, index) in columns" :key="'set'+index">
                                    <input class="form-check-input" :checked="item.show" type="checkbox" :value="item.show" :id="'settings-show-' + index" @click="columns[index].show = !columns[index].show">
                                    <label class="form-check-label" :for="'settings-show-' + index">{{ item.name }}</label>
                                </li>
                            </ul>
                        </div>
                    </div>
                    <div class="table-responsive"  ref="mainTable">
                        <div class="table">
                            <thead ref="tablehead">
                                <tr :class="{ondrag: draggableCol}">
                                    <th v-for="(col, index) in columns" :style="'width: ' + col.width + 'px;' + ' order:' + col.index" :key="'col'+index" v-show="col.show">
                                        <div class="th-inner" :data-index="col.index" @mousedown="moveColumn(col.index)">
                                            {{ col.name }}
                                        </div>
                                        <div class="resizer" @mousedown="startMove($event, index)"></div>
                                    </th>
                                </tr>
                            </thead>
                            <draggable
                                tag="tbody"
                                :list="tableData"
                                handle=".handle"
                                item-key="element.id"
                                @start="dragging = true"
                                @end="dragging = false;"
                            >
                                <template #item="{ element, index }">
                                    <tr>
                                        <td v-show="columns[0].show" :style="'width: ' + columns[0].width + 'px;' + 'order: ' + columns[0].index" class="handle" :class="{dragged: dragIndex == 0}">
                                            <label>{{ columns[0].name }}</label>
                                            <div><i class="icon icon-handle"></i> 
                                            <span>{{ index + 1}}</span></div>
                                        </td>
                                        <td v-show="columns[1].show" :style="'width: ' + columns[1].width + 'px;' + 'order: ' + columns[1].index" :class="{dragged: dragIndex == 1}">
                                            <label>{{ columns[1].name }}</label>
                                            <i class="icon icon-dots" @click="showModalWindow($event, index)"></i>
                                        </td>
                                        <td v-show="columns[2].show" :style="'width: ' + columns[2].width + 'px;' + 'order: ' + columns[2].index" :class="{dragged: dragIndex == 2}">
                                            <label>{{ columns[2].name }}</label>
                                            <vSelect
                                                :options="getOptionsNames(nameOptions)"
                                                :clearable="false"
                                                v-model="element.name"
                                                @open="blockHorizontalScroll(true)"
                                                @close="blockHorizontalScroll(false)"
                                                :appendToBody="true"
                                            >
                                                <template #no-options>Не найдено</template>
                                            </vSelect>
                                        </td>
                                        <td v-show="columns[3].show" :style="'width: ' + columns[3].width + 'px;' + 'order: ' + columns[3].index" :class="{dragged: dragIndex == 3}">
                                            <label>{{ columns[3].name }}</label>
                                            <input type="text" class="form-control" v-model="element.price" />
                                        </td>
                                        <td v-show="columns[4].show" :style="'width: ' + columns[4].width + 'px;' + 'order: ' + columns[4].index" :class="{dragged: dragIndex == 4}">
                                            <label>{{ columns[4].name }}</label>
                                            <input type="text" class="form-control" v-model="element.count" />
                                        </td>
                                        <td v-show="columns[5].show" :style="'width: ' + columns[5].width + 'px;' + 'order: ' + columns[5].index" :class="{dragged: dragIndex == 5}">
                                            <label>{{ columns[5].name }}</label>
                                            <vSelect
                                                :options="getOptionsNames(productOptions)"
                                                :clearable="false"
                                                v-model="element.productname"
                                                @open="blockHorizontalScroll(true)"
                                                @close="blockHorizontalScroll(false)"
                                                :appendToBody="true"
                                            >
                                                <template #no-options>Не найдено</template>
                                            </vSelect>
                                        </td>
                                        <td v-show="columns[6].show" :style="'width: ' + columns[6].width + 'px;' + 'order: ' + columns[6].index" :class="{dragged: dragIndex == 6}">
                                            <label>{{ columns[6].name }}</label>
                                            {{ element.count * element.price }}
                                        </td>
                                    </tr>
                                </template>
                            </draggable>
                        </div>
                    </div>
                    <div class="total">
                        <div class="total-block">
                            <div class="total-row">
                                <span class="text">Сумма:</span>
                                <span class="result">{{ total }} руб</span>
                            </div>
                            <div class="total-row">
                                <span class="text">Кол-во:</span>
                                <span class="result">{{ count }} шт</span>
                            </div>
                            <div class="total-row">
                                <span class="text">Общий вес:</span>
                                <span class="result">{{ weight }} кг</span>
                            </div>
                        </div>
                        <div class="total-block total-result">
                            <div class="total-row">
                                <span class="text">Общая сумма:</span>
                                <span class="result">{{ total }} руб</span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="page" v-show="page == 2">
                <div class="mainstyle article">
                    <p>Дополнительных расходов нет</p>
                </div>
            </div>
        </div>
        <div class="modalWindow" v-show="modalDelete.show" :style="'left: ' + modalDelete.posx + 'px; top: ' + modalDelete.posy + 'px;'" ref="modalDeleteWindow">
            <span @click="deleteRow(modalDelete.index)">Удалить</span>
        </div>
    </div>
</template>

<script>
import draggable from 'vuedraggable'
import vSelect from "vue-select";

export default {
    name: "App",
    components: {
        draggable,
        vSelect
    },
    data() {
        return {
            page: 1,
            showSettings: 0,
            tableData: [
                {
                    name: "Мраморный щебень фр. 2-5 мм, 25кг",
                    id: 0,
                    price: 123,
                    count: 11,
                    weight: 43,
                    productname: "Мраморный щебень",
                },
                {
                    name: "Мраморный щебень фр. 2-5 мм, 25кг",
                    id: 1,
                    price: 234,
                    count: 12,
                    weight: 45,
                    productname: "Мраморный щебень",
                },
                {
                    name: "Мраморный щебень фр. 2-5 мм, 25кг",
                    id: 2,
                    price: 567,
                    count: 13,
                    weight: 74,
                    productname: "Мраморный щебень",
                },
                {
                    name: "Мраморный щебень фр. 2-5 мм, 25кг",
                    id: 3,
                    price: 89,
                    count: 14,
                    weight: 149,
                    productname: "Мраморный щебень",
                },
                {
                    name: "Мраморный щебень фр. 2-5 мм, 25кг (белый)",
                    id: 4,
                    price: 99,
                    count: 15,
                    weight: 11,
                    productname: "Мраморный щебень",
                }
            ],
            drag: false,
            nameOptions: [
                {
                    name: "Мраморный щебень фр. 2-5 мм, 25кг",
                    id: 0
                },{
                    name: "Мраморный щебень фр. 2-5 мм, 25кг (белый)",
                    id: 1
                },{
                    name: "Мраморный щебень фр. 2-5 мм, 25кг (вайт)",
                    id: 2
                },{
                    name: "Мраморный щебень фр. 2-5 мм, 25кг, возрат",
                    id: 3
                },{
                    name: "Мраморный щебень фр. 2-5 мм, 1т",
                    id: 4
                }
            ],
            productOptions: [
                {
                    name: "Мраморный щебень",
                },{
                    name: "Мраморный щебень 2",
                },{
                    name: "Мраморный щебень 3",
                }
            ],
            columns: [
                {
                    name: "Номер строки",
                    width: 60,
                    min: 55,
                    show: true,
                    id: 0,
                    index: 0,
                },{
                    name: "Действие",
                    width: 30,
                    min: 30,
                    show: true,
                    id: 1,
                    index: 1,
                },{
                    name: "Наименование единицы",
                    width: 600,
                    min: 60,
                    show: true,
                    id: 2,
                    index: 2,
                },{
                    name: "Цена",
                    width: 200,
                    min: 60,
                    show: true,
                    id: 3,
                    index: 3,
                },{
                    name: "Кол-во",
                    width: 200,
                    min: 60,
                    show: true,
                    id: 4,
                    index: 4,
                },{
                    name: "Название товара",
                    width: 360,
                    min: 60,
                    show: true,
                    id: 5,
                    index: 5,
                },{
                    name: "Итого",
                    width: 160,
                    min: 40,
                    show: true,
                    id: 6,
                    index: 6,
                },

            ],
            draggableRow: false,
            draggableCol: false,
            dragIndex: -1,
            dragColIndex: -1,
            startOffset: 0,
            selectedCol: null,
            onInput: false,
            scrollTable: false,
            modalDelete: {
                show: false,
                posx: 20,
                posy: 15,
                index: -1,
            },
            headers: ["id", "name", "sport"],
            list: [
                { id: 1, name: "Abby", sport: "basket" },
                { id: 2, name: "Brooke", sport: "foot" },
                { id: 3, name: "Courtenay", sport: "volley" },
                { id: 4, name: "David", sport: "rugby" }
            ],
        }
    },
    computed: {
        total() {
            var result = 0;
            if (this.tableData) {
                this.tableData.forEach(element => {
                    result += parseInt(element.price) * parseInt(element.count);
                });
            }
            return result
        },
        count() {
            var result = 0;
            if (this.tableData) {
                this.tableData.forEach(element => {
                    result += parseInt(element.count);
                });
            }
            return result
        },
        weight() {
            var result = 0;
            if (this.tableData) {
                this.tableData.forEach(element => {
                    result += parseInt(element.weight);
                });
            }
            return result
        },
    },
    methods: {
        getOptionsNames(opt) {
            let newArr = [];
            opt.forEach(element => {
                newArr.push(element.name)
            });
            return newArr;
        },
        changePage(index) {
            if (index != this.page) {
                this.page = index;
            }
        },
        changeSettings(index) {
            if (index == 0 & this.showSettings == 0) {
                this.showSettings = 1;
                return
            }
            if (index == 1 & this.showSettings == 1) {
                this.showSettings = 0;
                return
            }
            this.showSettings = index;
        },
        addNewRow() {
            var newElement = {
                name: "Без названия",
                price: 0,
                count: 1,
                weight: 1,
            }
            this.tableData.push(newElement);
            this.sendDataToBackend(newElement, 'addNewRow')
        },
        move(e) {
            if (this.draggableRow) {
                let newWidth = this.startOffset + e.pageX;
                let minWidth = this.columns[this.selectedCol].min;
                
                if (!minWidth) {
                    minWidth = 0
                }
                if (newWidth <= minWidth) {
                    
                    newWidth = minWidth;
                }
                let colObj = this.columns[this.selectedCol];
                colObj.width = newWidth;
                this.columns[this.selectedCol] = colObj;
                
            }
            if (this.draggableCol) {
                if (e.target.className === "th-inner") {
                    var targetElement = e.target;
                    var targetIndex = parseInt(targetElement.dataset.index);
                    var oldIndex = this.dragColIndex;
                    var newIndex = targetIndex;

                    if (targetIndex == this.dragColIndex) {
                        return
                    }
                    var mouseX = e.clientX;
                    var elementWidth = targetElement.clientWidth;
                    var targetCoords = targetElement.getBoundingClientRect();
                    var pixels = Math.abs(mouseX - (targetCoords.x + elementWidth / 2))
                    if (pixels <= 5) {
                        var jj = this.columns.find(item => item.index == oldIndex);
                        var ee = this.columns.find(item => item.index == newIndex);
                        jj.index = newIndex;
                        ee.index = oldIndex;
                        this.draggableCol = false;
                        this.dragColIndex = -1;
                    }
                }
            }
        },
        startMove(e, index) {
            this.draggableRow = true;
            this.dragIndex = index;
            this.startOffset = this.columns[index].width - e.pageX;
            this.selectedCol = index;
        },
        stopMove(e) {
            if (this.draggableRow) {
                this.draggableRow = false;
                this.dragIndex = -1;
            }
            if (this.draggableCol) {
                this.draggableCol = false;
                this.dragColIndex = -1;
            }
            if (!this.$refs.modalDeleteWindow.contains(e.target)) {
                this.modalDelete.show = false;
                this.modalDelete.index = -1;
            }
            if (!this.$refs.modalA.contains(e.target)) {
                this.showSettings = 0;
            }
        },
        moveColumn(i) {
            this.dragColIndex = i;
            this.draggableCol = true;
        },
        scrollElement(e) {
            if (window.screen.width < 425) {
                return
            }
            let isScrollOnTable = e.target.closest('.table');
            if (isScrollOnTable) {
                e.preventDefault();
                if (this.onInput) {
                    return
                }
                if (e.deltaY > 0) {
                    this.$refs.mainTable.scrollBy(30, 0)
                } else {
                    this.$refs.mainTable.scrollBy(-30, 0)
                }
            }
        },
        displayToKey(event, element){
            element.name=event;
        },
        blockHorizontalScroll(bool) {
            this.onInput = bool;
        },
        showModalWindow(e, i) {
            if (this.modalDelete.index == i) {
                this.modalDelete.show = false;
                this.modalDelete.index = -1;
                return
            }
            this.modalDelete.show = true;
            this.modalDelete.index = i;
            this.modalDelete.posx = e.target.getBoundingClientRect().left - 5;
            this.modalDelete.posy = e.target.getBoundingClientRect().top + 20;
        },
        deleteRow() {
            this.sendDataToBackend(this.tableData[this.modalDelete.index], 'deleteRow')
            this.tableData.splice(this.modalDelete.index, 1);
           
            this.modalDelete.show = false;
            this.modalDelete.index = -1;      
        },
        saveAllChanges() {
            this.sendDataToBackend(this.tableData, 'newTableData')
        },
        sendDataToBackend(data, address) {
            data 
            address
            // Отправляем на бэк нужные данные
            // axios.post(`https://server/` + address, JSON.stringify(data)).then(() => { return });
        }
    },
    mounted() {
        document.addEventListener('mouseup', this.stopMove);
        document.addEventListener('mousemove', this.move);
    },
    beforeUnmount() {
        document.removeEventListener('mouseup', this.stopMove);
        document.removeEventListener('mousemove', this.stopMove);
    }
};
</script>