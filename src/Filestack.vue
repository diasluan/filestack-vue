<template>
  <div @click.prevent="onClickPick">
    <slot></slot>
  </div>
</template>

<script>
import filestack from 'filestack-js'
import VueTypes from 'vue-types'

export default {
  props: {
    apikey: VueTypes.string.def(''),
    file: VueTypes.objectOf(VueTypes.any).def(null),
    link: VueTypes.bool.def(false),
    mode: VueTypes.string.def('pick'),
    buttonText: VueTypes.string.def('Pick File'),
    buttonClass: VueTypes.string.def(''),
    options: VueTypes.objectOf(VueTypes.any).def({}),
    security: VueTypes.objectOf(VueTypes.any).def(null),
    cname: VueTypes.string.def(null)
  },
  methods: {
    onsuccess (result) {
      this.$emit('success', result)
    },
    onError (err) {
      this.$emit('error', err)
    },
    onClickPick () {
      this.initClient(
        this.mode,
        this.apikey,
        this.options,
        this.file,
        this.security,
        this.cname
      )
        .then((result) => this.onSuccess(result))
        .catch((err) => this.onError(err))
    },
    initClient (mode, apikey, options, file, security, cname) {
      const { url, handle } = options
      const client = filestack.init(apikey, security, cname)
      const ops = { ...options }
      delete ops.handle
      delete ops.url

      if (mode === 'transform') {
        return new Promise((resolve, reject) => {
          try {
            resolve(client.transform(url, ops))
          } catch (err) {
            reject(err)
          }
        })
      } else if (mode === 'retrieve') {
        return client.retrieve(handle, ops)
      } else if (mode === 'metadata') {
        return client.metadata(handle, ops)
      } else if (mode === 'storeUrl') {
        return client.storeURL(url, ops)
      } else if (mode === 'upload') {
        return client.upload(file, ops)
      } else if (mode === 'remove') {
        return client.remove(handle, security)
      }

      return client.pick(ops)
    }
  }
}
</script>

<style>
</style>
