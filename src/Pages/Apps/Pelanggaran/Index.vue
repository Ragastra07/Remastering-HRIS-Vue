<template>
    <Head>
        <title>Pelanggaran</title>
    </Head>
    <main>
        <div class="col-xl-12">
            <div class="card">
                <div class="card-header">
                    <button @click="buatBaruKategori" class="btn theme-bg4 text-white f-12 float-right" style="cursor:pointer; border:none; margin-right: 0px;" v-if="hasAnyPermission(['apps.list-pelanggaran.create'])"><i class="fa fa-plus"></i>Tambah</button>
                    <button class="btn btn-success dropdown-toggle float-right" type="button" id="dropdownMenuButton" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false" style="cursor:pointer; border:none;"><i class="fa fa-file-excel"></i> Excel</button>
                    <div class="dropdown-menu" aria-labelledby="dropdownMenuButton">
                        <a  :href="`/apps/list-pelanggaran/export`" target="_blank" class="dropdown-item">Export</a>
                        <!-- <button @click="importExcel" target="_blank" class="dropdown-item">Import</button> -->
                    </div>
                    <h5>Daftar Pelanggaran</h5>
                    <!-- <span class="d-block m-t-5">Page to manage the list of <code>{{ nama }}</code> violations</span> -->
                </div>
                <div class="card-block table-border-style">
                    <div class="table-responsive">
                        <div class="input-group mb-3">
                            <input
                            type="text"
                            class="form-control"
                            v-model="search"
                            placeholder="Cari berdasarkan catatan..."
                            @input="debouncedSearch"
                            >
                            <button
                                class="btn btn theme-bg5 text-white f-12"
                                style="margin-left: 10px;"
                                @click="handleSearch"
                            >
                                <i style="margin-left: 10px" class="fa fa-search me-2"></i>
                            </button>
                        </div>
                        <table class="table table-bordered table-hover">
                            <thead class="thead-light">
                                <tr>
                                    <th scope="col" class="text-center">#</th>
                                    <th scope="col" class="text-center">Nama Lengkap</th>
                                    <th scope="col" class="text-center">Catatan</th>
                                    <th scope="col" class="text-center">Tanggal</th>
                                    <th scope="col" class="text-center">Sanksi</th>
                                    <th scope="col" class="text-center">Status</th>
                                    <th scope="col" class="text-center" v-if="hasAnyPermission(['apps.list-pelanggaran.edit'])">Aksi</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="(list, index) in lists.data" :key="index">
                                    <td class="text-center">{{ index + lists.from }}</td>
                                    <td>{{ list.karyawan.nama_lengkap }}</td>
                                    <td>{{ list.catatan }}</td>
                                    <td>{{ list.tanggal }}</td>
                                    <td class="text-center" v-if="(list.tingkatan == null)"> </td>
                                    <td class="text-center" v-if="(list.tingkatan == 1)"><a class="label theme-bg9 text-white f-12" style="border-radius:10px">Teguran Lisan</a></td>
                                    <td class="text-center" v-if="(list.tingkatan == 2)"><a class="label theme-bg9 text-white f-12" style="border-radius:10px">Teguran Tertulis</a></td>
                                    <td class="text-center" v-if="(list.tingkatan == 3)"><a class="label theme-bg10 text-white f-12" style="border-radius:10px">SP 1</a></td>
                                    <td class="text-center" v-if="(list.tingkatan == 4)"><a class="label theme-bg7 text-white f-12" style="border-radius:10px">SP 2</a></td>
                                    <td class="text-center" v-if="(list.tingkatan == 5)"><a class="label theme-bg6 text-white f-12" style="border-radius:10px">SP 3</a></td>
                                    <td class="text-center" v-if="(list.status == null)"> </td>
                                    <td class="text-center" v-if="(list.status == 1)"><b style="color: rgb(240, 167, 9);">Diproses</b></td>
                                    <td class="text-center" v-if="(list.status == 2)"><b style="color: rgb(9, 240, 28);">Selesai</b></td>
                                    <td class="text-center" v-if="hasAnyPermission(['apps.list-pelanggaran.edit'])">
                                        <a @click="editData(list)" class="label theme-bg3 text-white f-12" style="cursor:pointer; border-radius:10px" v-if="hasAnyPermission(['apps.list-pelanggaran.edit'])"><i class="fa fa-pencil-alt"></i> Edit</a>
                                        <a @click="deleteData(list.id)" v-if="hasAnyPermission(['apps.pelanggaran.delete'])" class="label theme-bg3 text-white f-12" style="cursor:pointer; border-radius:10px; margin-left: 5px;"><i class="fa fa-trash-alt"></i> Delete</a>
                                    </td>
                                </tr>
                                <!-- jika data kosong -->
                                <tr v-if="lists.data[0] == undefined">
                                    <td colspan="7" class="text-center">
                                        <br>
                                        <i class="fa fa-file-excel fa-5x"></i><br><br>
                                            Data Kosong
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                        <div class="row mb-3" style="overflow-x:hidden">
                            <div class="col-md-4">
                                <label v-if="lists.data[0] != undefined" align="start">Showing {{ lists.from }} to {{ lists.to }} of {{ lists.total }} items</label>
                            </div>
                            <div class="col-md-8" style="float: right;">
                                <Pagination v-if="lists.data[0] != undefined" :links="lists.links" align="end"/>
                            </div>
                        </div>
                        <!-- <Link href="/apps/list-pelanggaran" class="btn btn-secondary shadow-sm" style="float: right; margin-right: 2px">Kembali</Link> -->
                    </div>
                </div>
            </div>
        </div>
        <!-- untuk modal untuk edit data -->
        <Teleport to="body">
            <modal :show="showModal" @close="showModal = false">
                <template #header>
                    <h5 class="modal-title">{{ judul }}</h5>
                </template>
                <template #body>
                    <div class="form-group mb-3">
                        <label class="col-form-label">Nama Lengkap :</label>
                        <VueMultiselect
                            v-model="karyawan_id"
                            :options="karyawan"
                            label="nama_lengkap"
                            track-by="id"
                            :allow-empty="false"
                            deselect-label="Can't remove this value"
                            placeholder="Pilih Karyawan"
                        ></VueMultiselect>
                    </div>
                    <div class="form-group mb-3">
                        <label class="col-form-label">Tanggal : </label>
                        <input type="date" class="form-control" placeholder="Date" v-model="tanggal" required>
                    </div>
                    <div class="form-group mb-3">
                        <label class="col-form-label">Catatan : </label>
                        <textarea type="text" class="form-control" placeholder="Notes" v-model="catatan" required></textarea>
                    </div>
                    <div class="row">
                        <div class="col-md-6">
                            <div class="form-group mb-0 mt-0">
                                <label class="col-form-label">Sanksi :</label>
                                <VueMultiselect
                                    v-model="tingkatan"
                                    :options="daftar_tingkatan"
                                    label="name"
                                    track-by="value"
                                    :allow-empty="false"
                                    deselect-label="Can't remove this value"
                                    placeholder="Pilih Sanksi"
                                ></VueMultiselect>
                            </div>
                        </div>

                        <div class="col-md-6">
                            <div class="form-group mb-0 mt-0">
                                <label class="col-form-label">Status :</label>
                                <VueMultiselect
                                    v-model="status"
                                    :options="daftar_status"
                                    label="name"
                                    track-by="value"
                                    :allow-empty="false"
                                    deselect-label="Can't remove this value"
                                    placeholder="Pilih Status"
                                ></VueMultiselect>
                            </div>
                        </div>
                    </div>
                </template>
                <template #footer>
					<form @submit.prevent="storeData">
						<button type="submit" v-show="!updateSubmit" class="btn btn-success text-white m-2">Simpan</button>
						<button type="button" v-show="updateSubmit" @click="updateData()" class="btn btn-warning text-white m-2">Update</button>
					</form>
                    <button
                        class="btn btn-secondary m-2" @click="tutupModal">Keluar</button>
                </template>
            </modal>
        </Teleport>
    </main>
</template>

<script>
    //import layout
    import LayoutApp from '../../../Layouts/App.vue';
    //import component pagination
    import Pagination from '../../../Components/Pagination.vue';
    //import Heade and Link from Inertia
    import { Head, Link  } from '@inertiajs/inertia-vue3';
    //import ref from vue
    import { ref } from 'vue';
    //import inertia adapter
    import { Inertia } from '@inertiajs/inertia';
    //import sweet alert2
    import Swal from 'sweetalert2';
    import Modal from '../../../Components/Modal.vue';
    import VueMultiselect from 'vue-multiselect';
    import 'vue-multiselect/dist/vue-multiselect.css';
    //import debounce [searching]
    import debounce from 'lodash/debounce';

    export default {
        //layout
        layout: LayoutApp,

        //register component
        components: {
            Head,
            Link,
            Pagination,
            Modal,
            VueMultiselect
        },

        //props
        props: {
            lists: Object,
            // nama: String,
            karyawan: Object,
        },

        setup(props) {
            //define state search
            const search = ref('' || (new URL(document.location)).searchParams.get('search'));
            const judul = ref(null);
            const updateSubmit = ref(false);
            const showModal = ref(false);
            const catatan = ref();
            const tingkatan = ref();
            const karyawan_id = ref();
            const tanggal = ref();
            const status = ref();
            const id_list = ref();

            const daftar_tingkatan = [
                { name: 'Teguran Lisan', value: 1 },
                { name: 'Teguran Tertulis', value: 2 },
                { name: 'SP 1', value: 3 },
                { name: 'SP 2', value: 4 },
                { name: 'SP 3', value: 5 },
            ];

            const daftar_status = [
                { name: 'Diproses', value: 1 },
                { name: 'Selesai', value: 2 },
            ];

            //define method search
            const handleSearch = () => {
                Inertia.get(`/apps/list-pelanggaran`, {
                    search: search.value,
                });
            }

            const debouncedSearch = debounce(handleSearch, 1000);

            //tampil modal
            const tampilModal = () => {
                showModal.value = true
            }

            //tutup modal
            const tutupModal = () => {
                showModal.value = false
            }

            //membuat kategori
            const buatBaruKategori = () =>{
                if(updateSubmit.value == true){
                    updateSubmit.value = !updateSubmit.value
                }
                judul.value = 'Tambah Pelanggaran'
                // id.value = null
                karyawan_id.value = null
                tanggal.value = null
                catatan.value = null
                tingkatan.value = null
                status.value = null
                tampilModal()
            }

            //edit data
            const editData = (list) => {

                if (updateSubmit.value == false) {
                    updateSubmit.value = !updateSubmit.value
                }

                //karyawan
                let data_karyawan = props.karyawan
                data_karyawan.forEach(data => {
                    if(list.karyawan_id == data.id){
                        karyawan_id.value = data
                    }
                })

                //tingkatan
                daftar_tingkatan.forEach(function (data) {
                    if(data.value == list.tingkatan){
                        tingkatan.value = data
                    }
                })

                //status tindakan
                daftar_status.forEach(function (data) {
                    if(data.value == list.status){
                        status.value = data
                    }
                })

                catatan.value = list.catatan
                tanggal.value = list.tanggal
                id_list.value = list.id
                // status.value = status.id
                showModal.value = true
            }

            const deleteData = (id) => {
                Swal.fire({
                    title: 'Apakah Anda Yakin?',
                    text: "Anda tidak akan dapat mengembalikan ini!",
                    icon: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Ya!',
                    cancelButtonText: 'Batal'
                })
                .then((result) => {
                    if (result.isConfirmed) {
                        Inertia.post(`/apps/pelatihan/${id}/delete`);
                        Swal.fire({
                            title: 'Sukses!',
                            text: 'Data Karyawan berhasil dihapus.',
                            icon: 'success',
                            timer: 2000,
                            showConfirmButton: false,
                        });
                    }
                })
            }

            const peringatan = () => {
                Swal.fire({
                    title: 'Mohon lengkapi isian!!',
                    width: 600,
                    padding: '3em',
                    color: '#716add',
                    backdrop: `
                        rgba(0,0,123,0.4)
                        left top
                        no-repeat
                    `
                })
            }

            //update data
            const updateData = () => {
                if(karyawan_id.value == null || tanggal.value == null || catatan.value == null || tingkatan.value == null  || status.value == null){
                    tutupModal();
                    peringatan();
                }else{
                    Inertia.put(`/apps/list-pelanggaran/${id_list.value}/update`, {
                        // id : id_list.value,
                        karyawan_id : karyawan_id.value.id,
                        catatan : catatan.value,
                        tanggal: tanggal.value,
                        tingkatan: tingkatan.value.value,
                        status: status.value.value,
                    },{
                        onSuccess: () => {
                            tutupModal()
                            //show success alert
                            Swal.fire({
                                title: 'Sukses!',
                                text: 'Pelanggaran berhasil ditambah.',
                                icon: 'success',
                                showConfirmButton: false,
                                timer: 2000
                            });
                        },
                    });

                }
            }

            //method "storeData"
            const storeData = () => {
                if(karyawan_id.value == null || tanggal.value == null || catatan.value == null || tingkatan.value == null  || status.value == null){
                    tutupModal();
                    peringatan();
                }
                else{
                    //send data to server
                    Inertia.post('/apps/list-pelanggaran/store', {
                        //data
                        karyawan_id : karyawan_id.value.id,
                        tanggal: tanggal.value,
                        catatan : catatan.value,
                        tingkatan: tingkatan.value.value,
                        status: status.value.value,

                    }, {
                        onSuccess: () => {
                            tutupModal()
                            //show success alert
                            Swal.fire({
                                title: 'Success!',
                                text: 'Data behasil ditambah.',
                                icon: 'success',
                                showConfirmButton: false,
                                timer: 2000
                            });
                        },
                    });
                }
            }

            //return
            return {
                search,
                handleSearch,
                editData, showModal, tampilModal, tutupModal, catatan, tingkatan, karyawan_id, tanggal, status, daftar_tingkatan, daftar_status,
                updateData, buatBaruKategori, judul, storeData, updateSubmit, peringatan, deleteData, debouncedSearch,
            }

        }
    }
</script>

<style>

</style>
