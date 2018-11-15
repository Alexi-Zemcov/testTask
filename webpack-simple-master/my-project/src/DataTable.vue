<template>
    <v-card>
        <v-card-title>
            Занимаемые должности
            <v-spacer></v-spacer>
            <v-text-field
                    v-model="searchName"
                    append-icon="search"
                    label="Поиск по сотруднику"
                    single-line
                    hide-details
            ></v-text-field>
        </v-card-title>

        <v-card-actions>
            <v-checkbox
                    :label="'Скрыть уволенных'"
                    v-model="checkboxShowFired"
            ></v-checkbox>
            <v-btn>Принять на должность</v-btn>
            <v-btn v-if="selected.length < 1" disabled>Снять с должности</v-btn>
            <v-btn v-else>{{ chooseButtonName(selected.length, 'Снять с должности', 'Снять с должностей') }}</v-btn>

        </v-card-actions>

        <v-data-table
                v-model="selected"
                :headers="headers"
                :items="workers"
                :pagination.sync="pagination"
                select-all
                item-key="name"
                class="elevation-1"
                rows-per-page-text="Строк на странице:"
                :search="searchName"
                :custom-filter="filterByName"
        >
            <template slot="headers" slot-scope="props">
                <tr>
                    <th>
                        <v-checkbox
                                :input-value="props.all"
                                :indeterminate="props.indeterminate"
                                primary
                                hide-details
                                @click.native="toggleAll"
                        ></v-checkbox>
                    </th>
                    <th
                            v-for="header in props.headers"
                            :key="header.text"
                            :class="[
                            'column sortable',
                            pagination.descending ? 'desc' : 'asc',
                            header.value === pagination.sortBy ? 'active' : ''
                         ]"
                            @click="changeSort(header.value)"
                            align="left"
                    >
                        {{ header.text }}
                        <v-icon small>arrow_upward</v-icon>
                    </th>
                </tr>
            </template>

            <template slot="items" slot-scope="props">
                <tr
                        v-if="(!props.item.fireDate || !checkboxShowFired)"
                        :active="props.selected"
                        :class="{error: props.item.fireDate}"
                >

                    <td>
                        <v-checkbox
                                v-model="props.selected"
                                primary
                                hide-details
                                v-if="!props.item.fireDate"
                        ></v-checkbox>
                    </td>
                    <td>{{ props.item.name }}</td>
                    <td>{{ props.item.companyName }}</td>
                    <td>{{ props.item.positionName }}</td>
                    <td>{{ makeDateFormatting(props.item.hireDate) }}</td>
                    <td>{{ makeDateFormatting(props.item.fireDate) }}</td>
                    <td>
                        <div v-if="!props.item.fireDate">
                            <v-edit-dialog
                                    large
                                    lazy
                                    persistent
                                    @save="save"
                                    @cancel="cancel"
                                    @open="open"
                                    @close="close"
                                    cancel-text="закрыть"
                                    save-text="сохранить"
                            >
                                <div>{{ `${props.item.salary} ₽ (${props.item.fraction}%)` }}</div>
                                <v-text-field
                                        slot="input"
                                        v-model="props.item.salary"
                                        :rules="[max7chars]"
                                        label="Редактирование"
                                        single-line
                                        counter
                                        autofocus
                                ></v-text-field>
                                <v-text-field
                                        slot="input"
                                        v-model="props.item.fraction"
                                        :rules="[max3chars]"
                                        label="Редактирование"
                                        single-line
                                        counter
                                        autofocus
                                ></v-text-field>

                            </v-edit-dialog>
                        </div>
                        <div v-else>{{ `${props.item.salary} ₽ (${props.item.fraction}%)` }}</div>
                    </td>


                    <td>
                        <div v-if="!props.item.fireDate">
                            <v-edit-dialog
                                    :return-value.sync="props.item.base"
                                    large
                                    lazy
                                    persistent
                                    @save="save"
                                    @cancel="cancel"
                                    @open="open"
                                    @close="close"
                                    cancel-text="закрыть"
                                    save-text="сохранить"
                            >
                                <div>{{ `${props.item.base} ₽` }}</div>
                                <v-text-field
                                        slot="input"
                                        v-model="props.item.base"
                                        :rules="[max7chars]"
                                        label="Редактирование"
                                        single-line
                                        counter
                                        autofocus
                                ></v-text-field>

                            </v-edit-dialog>
                        </div>
                        <div v-else>{{ `${props.item.base} ₽` }}</div>
                    </td>
                    <td>
                        <div v-if="(!props.item.fireDate)">
                            <v-edit-dialog
                                    :return-value.sync="props.item.advance"
                                    large
                                    lazy
                                    persistent
                                    @save="save"
                                    @cancel="cancel"
                                    @open="open"
                                    @close="close"
                                    cancel-text="закрыть"
                                    save-text="сохранить"
                            >
                                <div>{{ `${props.item.advance} ₽` }}</div>
                                <v-text-field
                                        slot="input"
                                        v-model="props.item.advance"
                                        :rules="[max7chars]"
                                        label="Редактирование"
                                        single-line
                                        counter
                                        autofocus
                                ></v-text-field>
                            </v-edit-dialog>
                        </div>
                        <div v-else>{{ `${props.item.advance} ₽` }}</div>
                    </td>
                    <td>
                        <div v-if="!props.item.fireDate">
                            <v-checkbox
                                    v-model="props.item.byHours"
                                    hide-details
                                    color="gray"
                            ></v-checkbox>
                        </div>
                        <div v-else>
                            <v-checkbox
                                    v-model="props.item.byHours"
                                    hide-details
                                    color="gray"
                                    readonly
                            ></v-checkbox>
                        </div>
                    </td>

                </tr>
            </template>

            <v-alert slot="no-results" :value="true" color="error" icon="warning">
                Ваш запрос "{{ searchName }}" не дал результатов.
            </v-alert>

        </v-data-table>


        <v-snackbar v-model="snack" :timeout="3000" :color="snackColor">
            {{ snackText }}
            <v-btn flat @click="snack = false">Закрыть</v-btn>
        </v-snackbar>

    </v-card>
</template>

<script>
    export default {
        data: () => ({

            //мои переменные
            checkboxShowFired: false,

            //изменение данных в таблице
            snack: false,
            snackColor: '',
            snackText: '',
            max7chars: v => v.length <= 7 || 'Не должно превыщать 7 символов!',
            max3chars: v => v.length <= 3 || 'Не должно превыщать 3 символа!',

            //эти шли с шаблоном таблицы
            searchName: '',
            pagination: {
                sortBy: 'Name',
                rowsPerPage: 25
            },
            selected: [],

            headers: [
                {text: 'Сотрудник', value: 'name'},
                {text: 'Компания', value: 'companyName'},
                {text: 'Должность', value: 'positionName'},
                {text: 'Дата приёма', value: 'hireDate'},
                {text: 'Дата увольнения', value: 'fireDate'},
                {text: 'Ставка', value: 'salary * fraction'}, //??? Сотрировка будет по ЗП умноженной на ставку
                {text: 'База', value: 'base'},
                {text: 'Аванс', value: 'advance'},
                {text: 'Почасовая', value: 'byHours'},
            ],
            workers: [
                {
                    name: 'Джордж Вашингтон',
                    companyName: 'ООО "Синергия"',
                    positionName: 'Первый президент США',
                    hireDate: '1789-04-30',
                    fireDate: '1797-03-04',
                    salary: 1000,
                    fraction: 100,
                    base: 1000,
                    advance: 1000,
                    byHours: false
                },
                {
                    name: 'Ричард I Львиное Сердце',
                    companyName: 'ООО "Синергия"',
                    positionName: 'Король Англии',
                    hireDate: '1189-01-01',
                    fireDate: '1199-05-17',
                    salary: 1000,
                    fraction: 100,
                    base: 1000,
                    advance: 1000,
                    byHours: true
                },
                {
                    name: 'Джейсон Стейтем',
                    companyName: 'ООО "Синергия"',
                    positionName: 'Бейкон',
                    hireDate: '1998-09-01',
                    fireDate: null,
                    salary: 1000,
                    fraction: 100,
                    base: 1000,
                    advance: 1000,
                    byHours: false
                },
                {
                    name: 'Тарантино К. Дж.',
                    companyName: 'ООО "Синергия"',
                    positionName: 'Джимми',
                    hireDate: '1994-04-15',
                    fireDate: null,
                    salary: 1000,
                    fraction: 100,
                    base: 1000,
                    advance: 1000,
                    byHours: false
                },
                {
                    name: 'Камбербэтч Б.',
                    companyName: 'ООО "Синергия"',
                    positionName: 'Смауг',
                    hireDate: '1000-01-01',
                    fireDate: '2941-10-09',
                    salary: 1000,
                    fraction: 100,
                    base: 1000,
                    advance: 1000,
                    byHours: false
                },
                {
                    name: 'Крузенштерн И. Ф.',
                    companyName: 'ООО "Синергия"',
                    positionName: 'Человек и пароход',
                    hireDate: '1770-11-08',
                    fireDate: null,
                    salary: 1000,
                    fraction: 100,
                    base: 1000,
                    advance: 1000,
                    byHours: false
                },
                {
                    name: 'Бендер С. Р.',
                    companyName: '"Planet Express"',
                    positionName: 'Робот-сгибальщик',
                    hireDate: '2997-03-27',
                    fireDate: null,
                    salary: 1000,
                    fraction: 100,
                    base: 1000,
                    advance: 1000,
                    byHours: true
                }
            ]
        }),
        methods: {
            //Шаблонные для выбора и сортировки данных в таблице
            toggleAll() {
                if (this.selected.length) {
                    this.selected = []
                } else {
                    this.selected = this.workers.slice()
                }
            },
            changeSort(column) {
                if (this.pagination.sortBy === column) {
                    this.pagination.descending = !this.pagination.descending
                } else {
                    this.pagination.sortBy = column;
                    this.pagination.descending = false;
                }
            },
            //Шаблонные для редактирования данных в таблице
            save() {
                this.snack = true;
                this.snackColor = 'success';
                this.snackText = 'Данные сохранены';
            },
            cancel() {
                this.snack = true;
                this.snackColor = 'error';
                this.snackText = 'Закрыт диалог редактирования';
            },
            open() {
                this.snack = true;
                this.snackColor = 'info';
                this.snackText = 'Открыт диалог редактирования';
            },
            close() {
                console.log('Dialog closed')
            },

            //Изменяет подпись кнопки
            chooseButtonName(inputValue, label1, label2) {
                if (typeof inputValue !== 'number') {
                    console.log(`Error input of the method "chooseButtonName"`);
                    return null
                }
                return inputValue <=1 ? label1: label2;
            },
            makeDateFormatting(date) {
                if ((!date) || (typeof date !== 'string')) {
                    return null
                }
                const arrDate = date.split('-');
                return `${arrDate[2]}.${arrDate[1]}.${arrDate[0]}`;
            },
            filterByName(items, search, filter) {
                const searchFormatted = search.toString().toLowerCase();
                return items.filter(row => filter(row["name"], searchFormatted));
            }
        }
    }
</script>