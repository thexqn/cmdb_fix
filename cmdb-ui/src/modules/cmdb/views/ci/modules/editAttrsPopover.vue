<template>
  <a-popover trigger="click" v-model="visible" placement="leftBottom" @visibleChange="visibleChange">
    <template slot="content">
      <AttributesTransfer
        :dataSource="attrList"
        :targetKeys="selectedAttrList"
        :showDefaultAttr="true"
        @setTargetKeys="setTargetKeys"
        @changeSingleItem="changeSingleItem"
        @handleSubmit="handleSubmit"
        :fixedList="fixedList"
        @setFixedList="setFixedList"
      />
    </template>
    <slot>
      <div :style="{ height: '100%', width: '30px', float: 'right', borderLeft: '1px solid #e8eaec' }">
        <a-icon :style="{ margin: '13px 0 0 10px ' }" type="control" />
      </div>
    </slot>
  </a-popover>
</template>

<script>
import AttributesTransfer from '../../../components/attributesTransfer'
import { subscribeCIType, getSubscribeAttributes } from '@/modules/cmdb/api/preference'
import { getCITypeAttributesByName } from '@/modules/cmdb/api/CITypeAttr'
import { CI_DEFAULT_ATTR } from '@/modules/cmdb/utils/const.js'

export default {
  name: 'EditAttrsPopover',
  components: { AttributesTransfer },
  props: {
    typeId: {
      type: Number,
      default: null,
    },
  },
  data() {
    return {
      visible: false,
      attrList: [],
      selectedAttrList: [],
      selectedKeys: [],
      fixedList: [],
    }
  },
  methods: {
    visibleChange(open) {
      if (open) {
        this.getAttrs()
      }
    },
    getAttrs() {
      const updatedByKey = CI_DEFAULT_ATTR.UPDATE_USER
      const updatedAtKey = CI_DEFAULT_ATTR.UPDATE_TIME

      getCITypeAttributesByName(this.typeId).then((res) => {
        const attributes = res.attributes.filter((item) => ![updatedByKey, updatedAtKey].includes(item.name))
        ;[updatedByKey, updatedAtKey].map((key) => {
          attributes.push({
            alias: key,
            name: key,
            id: key
          })
        })

        getSubscribeAttributes(this.typeId).then((_res) => {
          const selectedAttrList = _res.attributes.map((item) => item.id.toString())

          const attrList = attributes.map((item) => {
            return {
              key: item.id.toString(),
              title: item.alias || item.name,
              name: item.name,
            }
          })

          this.attrList = attrList
          this.selectedAttrList = selectedAttrList
          this.fixedList = _res.attributes.filter((item) => item.is_fixed).map((item) => item.id.toString())
          this.visible = true
        })
      })
    },

    handleSubmit() {
      if (this.selectedAttrList.length) {
        const customAttr = []
        const defaultAttr = []
        this.selectedAttrList.map((attr) => {
          if ([CI_DEFAULT_ATTR.UPDATE_USER, CI_DEFAULT_ATTR.UPDATE_TIME].includes(attr)) {
            defaultAttr.push(attr)
          } else {
            customAttr.push(attr)
          }
        })
        const selectedAttrList = [...customAttr, ...defaultAttr]

        subscribeCIType(
          this.typeId,
          selectedAttrList.map((item) => {
            return [item, !!this.fixedList.includes(item)]
          })
        ).then(() => {
          this.$message.success(this.$t('cmdb.components.subSuccess'))
          this.visible = false
          this.$emit('refresh')
        })
      } else {
        this.$message.error(this.$t('cmdb.ci.tips4'))
      }
    },
    setTargetKeys(targetKeys) {
      this.selectedAttrList = targetKeys
    },
    changeSingleItem(item) {
      const idx = this.selectedAttrList.findIndex((key) => key === item.key)
      if (idx > -1) {
        this.selectedAttrList.splice(idx, 1)
      } else {
        this.selectedAttrList.push(item.key)
      }
    },
    setFixedList(fixedList) {
      this.fixedList = fixedList
    },
  },
}
</script>
