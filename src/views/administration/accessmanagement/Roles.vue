<template>
  <b-card no-body :header="header">
    <b-card-body>
      <div id="customToolbar">
        <b-button size="md" variant="outline-primary" v-b-modal.createRoleModal>
          <span class="fa fa-plus"></span> {{ $t('admin.create_role') }}
        </b-button>
      </div>
      <bootstrap-table
        ref="table"
        :columns="columns"
        :data="data"
        :options="options"
      >
      </bootstrap-table>
    </b-card-body>
    <create-role-modal
      v-on:createRole="createRole"
      v-on:refreshTable="refreshTable"
    />
  </b-card>
</template>

<script>
import xssFilters from 'xss-filters';
import common from '../../../shared/common';
import i18n from '../../../i18n';
import CreateRoleModal from './CreateRoleModal';
import bootstrapTableMixin from '../../../mixins/bootstrapTableMixin';
import EventBus from '../../../shared/eventbus';
import ActionableListGroupItem from '../../components/ActionableListGroupItem';
import SelectPermissionModal from './SelectPermissionModal';
import permissionsMixin from '../../../mixins/permissionsMixin';
import { Switch as cSwitch } from '@coreui/vue';
import BInputGroupFormInput from '../../../forms/BInputGroupFormInput';

export default {
  props: {
    header: String,
  },
  mixins: [bootstrapTableMixin],
  components: {
    CreateRoleModal,
  },
  mounted() {
    this.data = [
      {
        name: 'Guest',
        description: 'Guest role',
        permissions: [
          Permissions.VIEW_PORTFOLIO,
          Permissions.VIEW_VULNERABILITY,
          Permissions.VIEW_BADGES,
        ],
      },
      {
        name: 'Planner',
        description: 'Planner role',
        permissions: [
          Permissions.VIEW_PORTFOLIO,
          Permissions.VIEW_VULNERABILITY,
          Permissions.VIEW_POLICY_VIOLATION,
          Permissions.VIEW_BADGES,
        ],
      },
      {
        name: 'Reporter',
        description: 'Reporter role',
        permissions: [
          Permissions.VIEW_PORTFOLIO,
          Permissions.VIEW_VULNERABILITY,
          Permissions.VIEW_POLICY_VIOLATION,
          Permissions.VIEW_BADGES,
        ],
      },
      {
        name: 'Maintainer',
        description: 'Maintainer role',
        permissions: [
          Permissions.BOM_UPLOAD,
          Permissions.PORTFOLIO_MANAGEMENT,
          Permissions.PORTFOLIO_MANAGEMENT_CREATE,
          Permissions.PORTFOLIO_MANAGEMENT_READ,
          Permissions.PORTFOLIO_MANAGEMENT_UPDATE,
          Permissions.PORTFOLIO_MANAGEMENT_DELETE,
          Permissions.VULNERABILITY_ANALYSIS,
          Permissions.VULNERABILITY_ANALYSIS_CREATE,
          Permissions.VULNERABILITY_ANALYSIS_READ,
          Permissions.VULNERABILITY_ANALYSIS_UPDATE,
          Permissions.POLICY_MANAGEMENT,
          Permissions.POLICY_MANAGEMENT_CREATE,
          Permissions.POLICY_MANAGEMENT_READ,
          Permissions.POLICY_MANAGEMENT_UPDATE,
          Permissions.POLICY_MANAGEMENT_DELETE,
        ],
      },
      {
        name: 'Developer',
        description: 'Developer role',
        permissions: [
          Permissions.BOM_UPLOAD,
          Permissions.VIEW_PORTFOLIO,
          Permissions.PORTFOLIO_MANAGEMENT_READ,
          Permissions.VIEW_VULNERABILITY,
          Permissions.VULNERABILITY_ANALYSIS_READ,
          Permissions.PROJECT_CREATION_UPLOAD,
        ],
      },
      {
        name: 'Owner',
        description: 'Owner role',
        permissions: [
          Permissions.ACCESS_MANAGEMENT,
          Permissions.ACCESS_MANAGEMENT_CREATE,
          Permissions.ACCESS_MANAGEMENT_READ,
          Permissions.ACCESS_MANAGEMENT_UPDATE,
          Permissions.ACCESS_MANAGEMENT_DELETE,
          Permissions.SYSTEM_CONFIGURATION,
          Permissions.SYSTEM_CONFIGURATION_CREATE,
          Permissions.SYSTEM_CONFIGURATION_READ,
          Permissions.SYSTEM_CONFIGURATION_UPDATE,
          Permissions.SYSTEM_CONFIGURATION_DELETE,
          Permissions.TAG_MANAGEMENT,
          Permissions.TAG_MANAGEMENT_DELETE,
        ],
      },
    ];
    EventBus.$on('admin:roles:rowUpdate', (index, row) => {
      this.$refs.table.updateRow({ index: index, row: row });
      this.$refs.table.expandRow(index);
    });
    EventBus.$on('admin:roles:rowDeleted', (index, row) => {
      this.refreshTable();
    });
  },
  beforeDestroy() {
    EventBus.$off('admin:roles:rowUpdate');
    EventBus.$off('admin:roles:rowDeleted');
  },
  data() {
    return {
      columns: [
        {
          title: this.$t('admin.role_name'),
          field: 'name',
          sortable: false,
          formatter(value) {
            return xssFilters.inHTMLData(common.valueWithDefault(value, ''));
          },
        },
        {
          title: this.$t('admin.description'),
          field: 'description',
          sortable: false,
          formatter(value, row) {
            return xssFilters.inHTMLData(common.valueWithDefault(value, ''));
          },
        },
      ],
      data: [],
      options: {
        search: true,
        showColumns: true,
        showRefresh: true,
        pagination: true,
        silentSort: false,
        sidePagination: 'client',
        queryParamsType: 'pageSize',
        pageList: '[10, 25, 50, 100]',
        pageSize: 10,
        icons: {
          refresh: 'fa-refresh',
        },
        detailView: true,
        detailViewIcon: false,
        detailViewByClick: true,
        detailFormatter: (index, row) => {
          return this.vueFormatter({
            i18n,
            template: `
                <b-row class="expanded-row">
                  <b-col sm="6">
                    <b-input-group-form-input id="input-role-name" :label="$t('admin.role_name')" input-group-size="mb-3"
                                              required="true" type="text" v-model="name" lazy="true" autofocus="true"
                                              v-debounce:750ms="updateRole" :debounce-events="'keyup'" />
                    <b-input-group-form-input id="input-role-description" :label="$t('admin.description')" input-group-size="mb-3"
                                              type="text" v-model="description" lazy="true" />
                    <b-form-group :label="this.$t('admin.permissions')">
                      <div class="list-group">
                        <span v-for="permission in permissions">
                          <actionable-list-group-item :value="permission.name" :delete-icon="true" v-on:actionClicked="removePermission(permission)"/>
                        </span>
                        <actionable-list-group-item :add-icon="true" v-on:actionClicked="$root.$emit('bv::show::modal', 'selectPermissionModal')"/>
                      </div>
                    </b-form-group>
                    <div style="text-align:right">
                       <b-button variant="outline-danger" @click="deleteRole">{{ $t('admin.delete_role') }}</b-button>
                    </div>
                  </b-col>
                </b-row>
              `,
            mixins: [permissionsMixin],
            components: {
              cSwitch,
              ActionableListGroupItem,
              SelectPermissionModal,
              BInputGroupFormInput,
            },
            data() {
              return {
                role: row,
                name: row.name,
                description: row.description,
                permissions: row.permissions,
                labelIcon: {
                  dataOn: '\u2713',
                  dataOff: '\u2715',
                },
              };
            },
            methods: {
              updateRole: function () {
                for (let i = 0; i < this.data.length; i++) {
                  if (this.data[i].name === this.name) {
                    this.data[i].name = this.name;
                    this.data[i].description = this.description;
                    break;
                  }
                }
                this.$toastr.s(this.$t('message.updated'));
              },
              deleteRole: function () {
                for (let i = 0; i < this.data.length; i++) {
                  if (this.data[i].name === this.name) {
                    this.data.splice(i, 1);
                    break;
                  }
                }
                this.$toastr.s(this.$t('admin.role_deleted'));
              },
              removePermission: function (permission) {
                for (let i = 0; i < this.data.length; i++) {
                  if (this.data[i].name === this.name) {
                    for (let j = 0; j < this.data[i].permissions.length; j++) {
                      if (
                        this.data[i].permissions[j].name === permission.name
                      ) {
                        this.data[i].permissions.splice(j, 1);
                        break;
                      }
                    }
                    break;
                  }
                }
                this.$toastr.s(this.$t('message.updated'));
              },
            },
          });
        },
        onExpandRow: this.vueFormatterInit,
        toolbar: '#customToolbar',
      },
    };
  },
  methods: {
    createRole(role) {
      this.data.push({
        name: role.name,
        description: role.description,
        permissions: [],
      });
      this.$toastr.s(this.$t('admin.role_created'));
    },
    refreshTable: function () {
      // do nothing
    },
  },
};
</script>
