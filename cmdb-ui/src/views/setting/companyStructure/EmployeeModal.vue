<template>
  <a-modal
    dialogClass="ops-modal"
    destroyOnClose
    width="810px"
    v-model="visible"
    :title="title"
    layout="inline"
    @cancel="close"
    :body-style="{ height: `${windowHeight - 320}px`, overflow: 'hidden', overflowY: 'scroll' }"
  >
    <a-form-model ref="employeeFormData" :model="employeeFormData" :rules="rules" :colon="false">
      <a-form-model-item
        ref="email"
        :label="$t('cs.companyStructure.email')"
        prop="email"
        :style="formModalItemStyle"
        v-if="attributes.findIndex((v) => v == 'email') !== -1"
      >
        <a-input v-model="employeeFormData.email" :placeholder="$t('cs.companyStructure.emailPlaceholder')" />
      </a-form-model-item>
      <a-form-model-item
        ref="username"
        :label="$t('cs.companyStructure.username')"
        prop="username"
        :style="formModalItemStyle"
        v-if="attributes.findIndex((v) => v == 'username') !== -1"
      >
        <a-input v-model="employeeFormData.username" :placeholder="$t('cs.companyStructure.usernamePlaceholder')" />
      </a-form-model-item>
      <a-form-model-item
        v-if="type === 'add'"
        ref="password"
        :label="$t('cs.companyStructure.password')"
        prop="password"
        :style="formModalItemStyle"
      >
        <a-input-password
          v-model="employeeFormData.password"
          :placeholder="$t('cs.companyStructure.passwordPlaceholder')"
        />
      </a-form-model-item>
      <a-form-model-item
        ref="nickname"
        :label="$t('cs.companyStructure.nickname')"
        prop="nickname"
        :style="formModalItemStyle"
        v-if="attributes.findIndex((v) => v == 'nickname') !== -1"
      >
        <a-input v-model="employeeFormData.nickname" :placeholder="$t('cs.companyStructure.nicknamePlaceholder')" />
      </a-form-model-item>
      <a-form-model-item
        :label="$t('cs.companyStructure.sex')"
        prop="sex"
        :style="formModalItemStyle"
        v-if="attributes.findIndex((v) => v == 'sex') !== -1"
      >
        <a-select v-model="employeeFormData.sex" :placeholder="$t('cs.companyStructure.sexPlaceholder')">
          <a-select-option value="男"> {{ $t('cs.companyStructure.male') }} </a-select-option>
          <a-select-option value="女"> {{ $t('cs.companyStructure.female') }} </a-select-option>
        </a-select>
      </a-form-model-item>
      <a-form-model-item
        ref="mobile"
        :label="$t('cs.companyStructure.mobile')"
        prop="mobile"
        :style="formModalItemStyle"
        v-if="attributes.findIndex((v) => v == 'mobile') !== -1"
      >
        <a-input v-model="employeeFormData.mobile" :placeholder="$t('cs.companyStructure.mobilePlaceholder')" />
      </a-form-model-item>
      <a-form-model-item
        ref="department_id"
        :label="$t('cs.companyStructure.departmentName')"
        prop="department_id"
        :style="formModalItemStyle"
        v-if="attributes.findIndex((v) => v == 'department_id') !== -1"
      >
        <DepartmentTreeSelect style="margin-top: 4px" v-model="employeeFormData.department_id" />
      </a-form-model-item>
      <a-form-model-item
        ref="position_name"
        :label="$t('cs.companyStructure.positionName')"
        prop="position_name"
        :style="formModalItemStyle"
        v-if="attributes.findIndex((v) => v == 'position_name') !== -1"
      >
        <a-input
          v-model="employeeFormData.position_name"
          :placeholder="$t('cs.companyStructure.positionNamePlaceholder')"
        />
      </a-form-model-item>
      <a-form-model-item
        ref="direct_supervisor_id"
        :label="$t('cs.companyStructure.selectDirectSupervisor')"
        prop="direct_supervisor_id"
        :style="formModalItemStyle"
        v-if="attributes.findIndex((v) => v == 'direct_supervisor_id') !== -1"
      >
        <EmployeeTreeSelect style="margin-top: 4px" v-model="employeeFormData.direct_supervisor_id" />
      </a-form-model-item>
      <a-form-model-item
        ref="work_region"
        :label="$t('cs.companyStructure.work_region')"
        prop="work_region"
        :style="formModalItemStyle"
        v-if="attributes.findIndex((v) => v == 'work_region') !== -1"
      >
        <a-select v-model="employeeFormData.work_region" :placeholder="$t('cs.companyStructure.workRegionPlaceholder')">
          <a-select-option value="china_mainland"> {{ $t('cs.companyStructure.china_mainland') }} </a-select-option>
          <a-select-option value="china_hk"> {{ $t('cs.companyStructure.china_hk') }} </a-select-option>
        </a-select>
      </a-form-model-item>
    </a-form-model>
    <template slot="footer">
      <a-button key="back" @click="close"> {{ $t('cancel') }} </a-button>
      <a-button type="primary" @click="employeeModalHandleOk"> {{ $t('confirm') }} </a-button>
    </template>
  </a-modal>
</template>
<script>
import { mapState } from 'vuex'
import _ from 'lodash'
import { postEmployee, putEmployee } from '@/api/employee'
import Bus from './eventBus/bus'
import EmployeeTreeSelect from '../components/employeeTreeSelect.vue'
import DepartmentTreeSelect from '../components/departmentTreeSelect.vue'
import moment from 'moment'
import { v4 as uuidv4 } from 'uuid'
export default {
  components: { EmployeeTreeSelect, DepartmentTreeSelect },
  data() {
    return {
      visible: false,
      employeeFormData: {},
      formModalItemStyle: { display: 'inline-block', width: '48%', margin: '0 7px 24px', overflow: 'hidden' },
      type: 'add',
      educational_experience: [],
      children_information: [],
      file_is_show: true,
      attributes: [],
    }
  },
  created() {
    Bus.$on('getAttributes', (attributes) => {
      this.attributes = attributes
    })
  },
  inject: ['provide_allTreeDepartment', 'provide_allFlatEmployees'],
  computed: {
    ...mapState({
      windowHeight: (state) => state.windowHeight,
    }),
    departemntTreeSelectOption() {
      return this.provide_allTreeDepartment()
    },
    allFlatEmployees() {
      return this.provide_allFlatEmployees()
    },
    title() {
      if (this.type === 'add') {
        return this.$t('cs.companyStructure.createEmployee')
      }
      return this.$t('cs.companyStructure.editEmployee')
    },
    rules() {
      return {
        email: [
          {
            required: true,
            whitespace: true,
            message: this.$t('cs.companyStructure.emailPlaceholder'),
            trigger: 'blur',
          },
          {
            pattern: /^[a-zA-Z0-9_.-]+@[a-zA-Z0-9-]+(\.[a-zA-Z0-9-]+)*\.[a-zA-Z0-9]{2,6}$/,
            message: this.$t('cs.companyStructure.emailFormatErr'),
            trigger: 'blur',
          },
          { max: 50, message: this.$t('cs.person.inputStrCountLimit', { limit: 50 }) },
        ],
        username: [
          {
            required: true,
            whitespace: true,
            message: this.$t('cs.companyStructure.usernamePlaceholder'),
            trigger: 'blur',
          },
          { max: 20, message: this.$t('cs.person.inputStrCountLimit', { limit: 20 }) },
        ],
        password: [
          {
            required: true,
            whitespace: true,
            message: this.$t('cs.companyStructure.passwordPlaceholder'),
            trigger: 'blur',
          },
        ],
        nickname: [
          {
            required: true,
            whitespace: true,
            message: this.$t('cs.companyStructure.nicknamePlaceholder'),
            trigger: 'blur',
          },
          { max: 20, message: this.$t('cs.person.inputStrCountLimit', { limit: 20 }) },
        ],
        mobile: [
          {
            pattern: /^(13[0-9]|14[01456879]|15[0-35-9]|16[2567]|17[0-8]|18[0-9]|19[0-35-9])\d{8}$/,
            message: this.$t('cs.companyStructure.mobileFormatErr'),
            trigger: 'blur',
          },
        ],
      }
    },
  },
  beforeDestroy() {
    Bus.$off('getAttributes')
  },
  methods: {
    async open(getData, type) {
      // 提交时去掉school, major, education, graduation_year, name, gender, birthday, parental_leave_left
      const { school, major, education, graduation_year, name, gender, birthday, parental_leave_left, ...newGetData } =
        getData
      const _getData = _.cloneDeep(newGetData)
      const { direct_supervisor_id } = newGetData
      if (direct_supervisor_id) {
        const _find = this.allFlatEmployees.find((item) => item.employee_id === direct_supervisor_id)
        _getData.direct_supervisor_id = `${_find.department_id}-${direct_supervisor_id}`
      } else {
        _getData.direct_supervisor_id = undefined
      }
      this.employeeFormData = _.cloneDeep(_getData)
      // if (type !== 'add' && this.employeeFormData.educational_experience.length !== 0) {
      //   this.educational_experience = this.employeeFormData.educational_experience
      // }
      this.children_information = this.formatChildrenInformationList() || [
        {
          id: uuidv4(),
          name: '',
          gender: undefined,
          birthday: null,
          parental_leave_left: 0,
        },
      ]
      this.educational_experience = this.formatEducationalExperienceList() || [
        {
          id: uuidv4(),
          school: '',
          major: '',
          education: undefined,
          graduation_year: null,
        },
      ]

      this.type = type
      this.visible = true
    },
    close() {
      this.$refs.employeeFormData.resetFields()
      this.educational_experience = [
        {
          school: '',
          major: '',
          education: undefined,
          graduation_year: null,
        },
      ]
      this.children_information = [
        {
          id: uuidv4(),
          name: '',
          gender: undefined,
          birthday: null,
          parental_leave_left: 0,
        },
      ]
      this.visible = false
    },
    formatChildrenInformationList() {
      let arr = []
      arr = this.employeeFormData.children_information ? this.employeeFormData.children_information : undefined
      if (arr && arr.length) {
        arr.forEach((item) => {
          item.id = uuidv4()
        })
        return arr
      }
      return null
    },
    formatEducationalExperienceList() {
      let arr = []
      arr = this.employeeFormData.educational_experience ? this.employeeFormData.educational_experience : undefined
      if (arr && arr.length) {
        arr.forEach((item) => {
          item.id = uuidv4()
        })
        return arr
      }
      return null
    },
    addEducation() {
      const newEducational_experience = {
        id: uuidv4(),
        school: '',
        major: '',
        education: undefined,
        graduation_year: null,
      }
      this.educational_experience.push(newEducational_experience)
    },
    removeEducation(removeId) {
      const _idx = this.educational_experience.findIndex((item) => item.id === removeId)
      if (_idx !== -1) {
        this.educational_experience.splice(_idx, 1)
      }
    },
    addChildren() {
      const newChildrenInfo = {
        id: uuidv4(),
        name: '',
        gender: undefined,
        birthday: null,
        parental_leave_left: 0,
      }
      this.children_information.push(newChildrenInfo)
    },
    removeChildren(removeId) {
      const _idx = this.children_information.findIndex((item) => item.id === removeId)
      if (_idx !== -1) {
        this.children_information.splice(_idx, 1)
      }
    },
    onChange(date, param, id) {
      // if (param === 'graduation_year') {
      //   if (date === null) {
      //     this.educational_experience[index].graduation_year = null
      //   } else {
      //     this.educational_experience[index].graduation_year = moment(date).format('YYYY-MM-DD')
      //   }
      // } else {
      //   if (date === null) {
      //     this.employeeFormData[param] = null
      //   } else {
      //     this.employeeFormData[param] = moment(date).format('YYYY-MM-DD')
      //   }
      // }
      if (date !== null) {
        if (param === 'graduation_year') {
          const _idx = this.educational_experience.findIndex((item) => item.id === id)
          this.educational_experience[_idx].graduation_year = moment(date).format('YYYY-MM')
        } else if (param === 'birthday') {
          const _idx = this.children_information.findIndex((item) => item.id === id)
          this.children_information[_idx].birthday = moment(date).format('YYYY-MM-DD')
        } else {
          this.employeeFormData[param] = moment(date).format('YYYY-MM-DD')
        }
      }
    },
    async employeeModalHandleOk() {
      if (this.attributes.includes('educational_experience')) {
        this.employeeFormData.educational_experience = this.educational_experience
      }
      if (this.attributes.includes('children_information')) {
        this.employeeFormData.children_information = this.children_information
      }
      // if (!this.employeeFormData.annual_leave) {
      //   this.employeeFormData.annual_leave = 0
      // }
      const getFormData = this.employeeFormData
      getFormData.direct_supervisor_id = getFormData.direct_supervisor_id
        ? (getFormData.direct_supervisor_id + '').includes('-')
          ? +getFormData.direct_supervisor_id.split('-')[1]
          : +getFormData.direct_supervisor_id
        : 0
      this.$refs.employeeFormData.validate(async (valid) => {
        if (valid) {
          if (this.type === 'add') {
            await postEmployee(getFormData)
          }
          if (this.type === 'edit') {
            await putEmployee(getFormData.employee_id, getFormData)
          }
          this.$message.success(this.$t('cs.companyStructure.opSuccess'))
          this.$emit('refresh')
          Bus.$emit('updataAllIncludeEmployees')
          this.close()
        } else {
          this.$message.warning(this.$t('cs.companyInfo.checkInputCorrect'))
          return false
        }
      })
    },
  },
}
</script>
<style lang="less" scoped>
.el-date-picker {
  width: 100%;
  height: 36px;
}
</style>
