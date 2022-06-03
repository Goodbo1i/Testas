<template>
<div >
  <vuci-form :uci-config="config" >
    <vuci-typed-section :type="section" :columns="columns" >
       <template #interface_name="{ s }" >
        <vuci-form-item-dummy :uci-section="s"  name="interface_name" />
      </template>
      <template #address="{ s }">
        <vuci-form-item-dummy :uci-section="s"  name="address" rules="ipaddr"/>
      </template>
      <template #netmask="{ s }">
        <vuci-form-item-dummy :uci-section="s"  name="netmask" rules="netmask4"/>
      </template>
      <template #action="{ s }">
        <a-button type="primary" size="small" :uci-section="s" style="margin-right: 5px" @click="editInterface(s['.name'])"> Edit </a-button>
        <a-popconfirm @confirm="del(s['.name'])" placement="left" :title="'Delete this interface?'">
          <a-button type="danger" size="small" :uci-section="s">Delete</a-button>
        </a-popconfirm>
      </template>
    </vuci-typed-section>
  </vuci-form>
  <template>
      <a-input v-model="interfaceName" placeholder="Interface Name" />
      <a-button type="primary" size="small" style="margin-top: 10px" @click="handleAdd">+ Add interface</a-button>
  </template>
  <a-modal  v-model="editModal" :width="800" >
      <vuci-form :uci-config="config" v-if="editModal">
        <div> {{ sectionToEdit }}</div>
        <vuci-named-section :name="sectionToEdit" v-slot="{ s }">
            <vuci-form-item-select label="Protocol"  name="proto" initial="static" :options="protocols" :uci-section="s"/>
            <vuci-form-item-input label="IP Address" :uci-section="s" name="address"  rules="ipaddr" required/>
            <vuci-form-item-input label="Netmask" :uci-section="s" name="netmask"  rules="netmask4" required/>
            <vuci-form-item-input label="Gateway"  name="gateway" :uci-section="s" />
            <vuci-form-item-list label="DNS" name="dns" :uci-section="s"/>
            <vuci-form-item-switch label="DHCP" name="dhcp" :initial="true" :uci-section="s"/>
          </vuci-named-section>
      </vuci-form>
      <template #footer><div/></template>
    </a-modal>
    </div>
</template>

<script>
export default {

  data () {
    return {
      config: 'example',
      section: 'interface',
      sections: [],
      interfaces: {},
      interfaceName: '',
      wrapperCol: { span: 14 },
      columns: [
        { name: 'interface_name', label: 'Interface name' },
        { name: 'address', label: 'IP address changed' },
        { name: 'netmask', label: 'Netmask changed' },
        { name: 'action', label: '#' }
      ],
      editModal: false,
      protocols: [
        ['dhcp', 'DHCP Client'],
        ['static', 'Static address']
      ],
      sectionToEdit: undefined
    }
  },
  methods: {
    editInterface (sectionName) {
      this.sectionToEdit = sectionName
      this.editModal = true
    },
    del (name) {
      this.$spin()
      this.$uci.del(this.config, name)
      this.$uci.save().then(() => {
        this.$uci.apply().then(() => {
          this.load()
          this.$spin(false)
        })
      })
    },
    add (name) {
      this.$spin()
      const sid = this.$uci.add(this.config, this.section)
      this.$uci.set(this.config, sid, 'interface_name', name)
      this.$uci.save().then(() => {
        this.$uci.apply().then(() => {
          this.load()
          this.$spin(false)
        })
      })
    },
    handleAdd () {
      if (!this.interfaceName) return
      if (this.interfaceName.match(/^[a-zA-Z0-9_]+$/) === null) {
        this.$message.error(this.$t('validator.uci'))
        return
      }
      for (let i = 0; i < this.interfaces.length; i++) {
        if (this.interfaces[i].name === this.interfaceName) {
          this.$message.error(this.$t('Name already used'))
          return
        }
      }
      this.add(this.interfaceName)
    },
    async load () {
      await this.$uci.load(this.config)
      const bbb = this.sections = this.$uci.sections(this.config, this.section)
      console.log(bbb)
      console.log(this.config)
      this.$forceUpdate()
      this.reload()
    },
    reload () {
      this.$forceUpdate()
    },
    filter (s) {
      return s['.name'] !== ''
    }
  },
  created () {
    this.load()
  }
}
</script>
