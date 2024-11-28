<template>
    <div class="page row">
        <!-- Phần tìm kiếm -->
        <div class="col-md-10">
            <InputSearch v-model="searchText" />
        </div>

        <!-- Phần danh bạ -->
        <div class="mt-3 col-md-6">
            <h4>
                Danh bạ
                <i class="fas fa-address-book"></i>
            </h4>
            <ContactList v-if="filteredContactsCount > 0" :contacts="filteredContacts"
                v-model:activeIndex="activeIndex" />
            <p v-else>Không có liên hệ nào.</p>

            <!-- Các nút điều khiển -->
            <div class="mt-3 row justify-content-around align-items-center">
                <button class="btn btn-sm btn-primary" @click="refreshList">
                    <i class="fas fa-redo"></i> Làm mới
                </button>
                <button class="btn btn-sm btn-success" @click="goToAddContact">
                    <i class="fas fa-plus"></i> Thêm mới
                </button>
                <button class="btn btn-sm btn-danger" @click="removeAllContacts">
                    <i class="fas fa-trash"></i> Xóa tất cả
                </button>
            </div>
        </div>

        <!-- Phần chi tiết liên hệ -->
        <div class="mt-3 col-md-6">
            <div v-if="activeContact">
                <h4>
                    Chi tiết Liên hệ
                    <i class="fas fa-address-card"></i>
                </h4>
                <ContactCard :contact="activeContact" />
                <router-link :to="{ name: 'contact.edit', params: { id: activeContact._id } }">
                    <span class="mt-2 badge badge-warning">
                        <i class="fas fa-edit"></i> Hiệu chỉnh
                    </span>
                </router-link>
            </div>
        </div>
    </div>
</template>

<script>
import ContactCard from "@/components/ContactCard.vue";
import InputSearch from "@/components/InputSearch.vue";
import ContactList from "@/components/ContactList.vue";
import ContactService from "@/services/contact.service";

export default {
    components: {
        ContactCard,
        InputSearch,
        ContactList,
    },
    data() {
        return {
            contacts: [],        // Danh sách liên hệ
            activeIndex: -1,     // Chỉ mục của liên hệ đang được chọn
            searchText: "",      // Văn bản tìm kiếm
        };
    },
    watch: {
        // Giám sát các thay đổi của biến searchText.
        searchText() {
            this.activeIndex = -1;  // Bỏ chọn phần tử khi tìm kiếm thay đổi
        },
    },
    computed: {
        // Chuyển các đối tượng contact thành chuỗi để tiện cho tìm kiếm
        contactStrings() {
            return this.contacts.map((contact) => {
                const { name, email, address, phone } = contact;
                return [name, email, address, phone].join("");
            });
        },
        // Trả về các contact có chứa thông tin cần tìm kiếm
        filteredContacts() {
            if (!this.searchText) return this.contacts;
            return this.contacts.filter((contact, index) =>
                this.contactStrings[index].includes(this.searchText)
            );
        },
        activeContact() {
            if (this.activeIndex < 0) return null;
            return this.filteredContacts[this.activeIndex];
        },
        filteredContactsCount() {
            return this.filteredContacts.length;
        },
    },
    methods: {
        // Lấy danh sách liên hệ
        async retrieveContacts() {
            try {
                this.contacts = await ContactService.getAll();
            } catch (error) {
                console.error("Error fetching contacts:", error);
            }
        },
        // Làm mới danh sách liên hệ
        refreshList() {
            this.retrieveContacts();
            this.activeIndex = -1;  // Reset activeIndex khi làm mới
        },
        // Xóa tất cả các liên hệ
        async removeAllContacts() {
            if (confirm("Bạn muốn xóa tất cả Liên hệ?")) {
                try {
                    await ContactService.deleteAll();
                    this.refreshList();  // Làm mới danh sách sau khi xóa
                } catch (error) {
                    console.error("Error deleting all contacts:", error);
                }
            }
        },
        // Điều hướng đến trang thêm mới liên hệ
        goToAddContact() {
            this.$router.push({ name: "contact.add" });
        },
    },
    mounted() {
        this.refreshList();
    },
};
</script>

<style scoped>
.page {
    text-align: left;
    max-width: 750px;
}
</style>
