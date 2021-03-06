<template>
  <b-modal id="projectDetailsModal" size="md" hide-header-close no-stacking :title="$t('message.project_details')">
    <b-form-group
      id="fieldset-1"
      :label="this.$t('message.project_name')"
      label-for="input-1"
      label-class="required">
      <b-form-input id="input-1" v-model="project.name" class="required" :readonly="this.isNotPermitted(PERMISSIONS.PORTFOLIO_MANAGEMENT)" trim />
    </b-form-group>
    <b-form-group
      id="fieldset-2"
      :label="this.$t('message.version')"
      label-for="input-2">
      <b-form-input id="input-2" v-model="project.version" :readonly="this.isNotPermitted(PERMISSIONS.PORTFOLIO_MANAGEMENT)" trim />
    </b-form-group>
    <b-form-group
      id="fieldset-3"
      :label="this.$t('message.description')"
      label-for="input-3">
      <b-form-textarea id="input-3" v-model="project.description" :readonly="this.isNotPermitted(PERMISSIONS.PORTFOLIO_MANAGEMENT)" trim />
    </b-form-group>
    <b-form-group
      id="fieldset-4"
      :label="this.$t('message.tags')"
      label-for="input-4">
      <vue-tags-input id="input-4" v-model="tag" :tags="tags" :add-on-key="addOnKeys"
                      :placeholder="$t('message.add_tag')" @tags-changed="newTags => this.tags = newTags"
                      :readonly="this.isNotPermitted(PERMISSIONS.PORTFOLIO_MANAGEMENT)"
                      style="max-width:none; background-color:transparent;"/>
    </b-form-group>

    <c-switch id="input-5" class="mx-1" color="primary" v-model="project.active" label :disabled="this.isNotPermitted(PERMISSIONS.PORTFOLIO_MANAGEMENT)" v-bind="labelIcon" /> {{$t('message.active')}}

    <template v-slot:modal-footer="{ cancel }">
      <b-button size="md" variant="outline-danger" @click="deleteProject()" v-permission="PERMISSIONS.PORTFOLIO_MANAGEMENT">{{ $t('message.delete') }}</b-button>
      <b-button size="md" variant="outline-primary" v-b-modal.projectPropertiesModal v-permission="PERMISSIONS.PORTFOLIO_MANAGEMENT">{{ $t('message.properties') }}</b-button>
      <b-button size="md" variant="outline-primary" v-b-modal.projectAddVersionModal v-permission="PERMISSIONS.PORTFOLIO_MANAGEMENT">{{ $t('message.add_version') }}</b-button>
      <b-button size="md" variant="secondary" @click="cancel()">{{ $t('message.close') }}</b-button>
      <b-button size="md" variant="primary" @click="updateProject()" v-permission="PERMISSIONS.PORTFOLIO_MANAGEMENT">{{ $t('message.update') }}</b-button>
    </template>
  </b-modal>
</template>

<script>
  import VueTagsInput from '@johmun/vue-tags-input';
  import { Switch as cSwitch } from '@coreui/vue';
  import permissionsMixin from "../../../mixins/permissionsMixin";
  import common from "../../../shared/common";

  export default {
    name: "ProjectDetailsModal",
    mixins: [permissionsMixin],
    components: {
      VueTagsInput,
      cSwitch
    },
    props: {
      project: Object
    },
    data() {
      return {
        tag: '', // The contents of a tag as its being typed into the vue-tag-input
        tags: [], // An array of tags bound to the vue-tag-input
        addOnKeys: [9, 13, 32, ':', ';', ','], // Separators used when typing tags into the vue-tag-input
        labelIcon: {
          dataOn: '\u2713',
          dataOff: '\u2715'
        },
      }
    },
    beforeUpdate() {
      if (this.tags.length === 0 && this.project && this.project.tags)  { // Prevents line from being executed when entering new tags
        this.project.tags.forEach((tag) => this.tags.push({text: tag.name}));
      }
    },
    methods: {
      updateProject: function() {
        this.$root.$emit('bv::hide::modal', 'projectDetailsModal');
        let url = `${this.$api.BASE_URL}/${this.$api.URL_PROJECT}`;
        let tagsNode = [];
        this.tags.forEach((tag) => tagsNode.push({name: tag.text}));
        this.axios.post(url, {
            uuid: this.project.uuid,
            name: this.project.name,
            version: this.project.version,
            description: this.project.description,
            tags: tagsNode,
            active: this.project.active
          }).then((response) => {
            this.$emit('projectUpdated', response.data);
            this.$toastr.s(this.$t('message.project_updated'));
          }).catch((error) => {
            this.$toastr.w(this.$t('condition.unsuccessful_action'));
        });
      },
      deleteProject: function() {
        this.$root.$emit('bv::hide::modal', 'projectDetailsModal');
        let url = `${this.$api.BASE_URL}/${this.$api.URL_PROJECT}/` + this.project.uuid;
        this.axios.delete(url).then((response) => {
          this.$toastr.s(this.$t('message.project_deleted'));
          this.$router.replace({ name: "Projects" });
        }).catch((error) => {
          this.$toastr.w(this.$t('condition.unsuccessful_action'));
        });
      }
    }
  }
</script>

<style lang="scss">
  @import "../../../assets/scss/vendors/vue-tags-input/vue-tags-input";
</style>
