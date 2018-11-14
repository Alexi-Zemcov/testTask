<template>
    <v-app>
        <v-content>
            <app-dataTable></app-dataTable>
        </v-content>

        <v-footer :fixed="fixed" app>
            <v-spacer></v-spacer>
            <span>&copy; 2018</span>
        </v-footer>
    </v-app>
</template>

<script>
    import DataTable from './DataTable.vue'
    export default {
        data() {
            return {fixed: false}
        },
        components: {
            'app-dataTable': DataTable
        }
    }
</script>
