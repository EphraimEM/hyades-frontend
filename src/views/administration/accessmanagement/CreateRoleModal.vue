<template>
  <b-modal
    id="createRoleModal"
    :title="$t('admin.create_role')"
    @hidden="resetModal"
    @shown="focusInput"
    @ok="createRole"
  >
    <form @submit.stop.prevent="createRole()">
      <b-input-group-form-input
        id="input-role-name"
        :label="$t('admin.role_name')"
        input-group-size="mb-3"
        required="true"
        type="text"
        v-model="name"
        lazy="true"
        autofocus="true"
      />
      <b-input-group-form-input
        id="input-role-description"
        :label="$t('admin.role_description')"
        input-group-size="mb-3"
        type="text"
        v-model="description"
        lazy="true"
      />
    </form>
  </b-modal>
</template>

<script>
export default {
  data() {
    return {
      name: '',
      description: '',
    };
  },
  methods: {
    createRole() {
      this.$axios
        .post(this.$api.BASE_URL + '/api/v1/role', {
          name: this.name,
          description: this.description,
        })
        .then((response) => {
          this.$toastr.s(this.$t('admin.role_created'));
          this.$emit('refreshTable');
          this.$bvModal.hide('createRoleModal');
        })
        .catch((error) => {
          this.$toastr.w(this.$t('condition.unsuccessful_action'));
        });
    },
    resetModal() {
      this.name = '';
      this.description = '';
    },
    focusInput() {
      const input = this.$refs.nameInput;
      input.focus();
    },
  },
};
</script>
