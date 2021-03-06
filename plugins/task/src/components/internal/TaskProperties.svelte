<script type="ts">
  // Copyright © 2020 Anticrm Platform Contributors.
  //
  // Licensed under the Eclipse Public License, Version 2.0 (the "License");
  // you may not use this file except in compliance with the License. You may
  // obtain a copy of the License at https://www.eclipse.org/legal/epl-2.0
  //
  // Unless required by applicable law or agreed to in writing, software
  // distributed under the License is distributed on an "AS IS" BASIS,
  // WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  //
  // See the License for the specific language governing permissions and
  // limitations under the License.
  import { onDestroy } from 'svelte'
  import { Ref, Class, Obj, Backlinks } from '@anticrm/core'
  import task, { Task } from '../..'
  import core from '@anticrm/platform-core'
  import { getPresentationService, find, query } from '../../utils'
  import { AttrModel, ClassModel } from '@anticrm/presentation'

  import Properties from '@anticrm/presentation/src/components/internal/Properties.svelte'
  import AttributeEditor from '@anticrm/presentation/src/components/AttributeEditor.svelte'

  export let _class: Ref<Class<Obj>>
  export let object: Task

  let model: ClassModel | undefined
  let title: AttrModel | undefined

  $: getPresentationService()
    .then((service) => service.getClassModel(_class, core.class.VDoc))
    .then((m) => {
      title = m.getAttribute('title')
      model = m.filterAttributes(['title'])
    })

  let backlinks: Backlinks[]
  let unsubscribe: () => void
  $: {
    if (unsubscribe) {
      unsubscribe()
    }
    unsubscribe = query(core.class.Backlinks, { _objectId: object._id }, (docs) => {
      backlinks = docs
    })
  }

  onDestroy(() => {
    if (unsubscribe) unsubscribe()
  })
</script>

{#if model && title}
  <div>
    <!-- <StringPresenter class="caption-1" :attribute="name" v-model="object[name.key]" /> -->
    <div class="caption-1">
      <AttributeEditor attribute="{title}" bind:value="{object.title}" />
    </div>
  </div>

  <!-- <Properties _class={task.class.Task} excludeAttributes={['title']} /> -->
  <Properties model="{model}" bind:object />

  <div class="caption-2">Backlinks</div>

  {#each backlinks as backlink (backlink._id)}
    <div>{JSON.stringify(backlink)}</div>
  {/each}
{/if}
