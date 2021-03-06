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
  import { MessageNode, Property, Ref, Space, VDoc } from '@anticrm/core'
  import core from '@anticrm/platform-core'
  import ReferenceInput from '@anticrm/presentation/src/components/refinput/ReferenceInput.svelte'
  import ScrollView from '@anticrm/sparkling-controls/src/ScrollView.svelte'
  import { onDestroy } from 'svelte'
  import chunter, { Comment, Message } from '../..'
  import { getChunterService, getCoreService, query } from '../../utils'
  import CommentComponent from './Comment.svelte'

  const coreService = getCoreService()
  const chunterService = getChunterService()

  export let space: Ref<Space>

  let spaceName: string
  let messages: Message[] = []
  let unsubscribe: () => void

  $: {
    if (unsubscribe) {
      unsubscribe()
    }
    unsubscribe = query(chunter.class.Message, { _space: space }, (docs) => {
      messages = docs
    })

    // TODO: use Titles index instead of getting the whole Space object
    coreService.then((service) => service.findOne(core.class.Space, { _id: space })).then((spaceObj) => (spaceName = spaceObj ? '#' + spaceObj.name : ''))
  }

  onDestroy(() => {
    if (unsubscribe) unsubscribe()
  })

  function createMessage(message: MessageNode) {
    if (message) {
      chunterService.then((chunterService) => {
        const parsedMessage = chunterService.createMissedObjects(message)
        coreService.then((coreService) => {
          const comment: Omit<Comment, '__embedded'> = {
            _class: chunter.class.Comment,
            _createdOn: Date.now() as Property<number, Date>,
            _createdBy: 'john.appleseed@gmail.com' as Property<string, string>,
            message: parsedMessage
          }
          const newMessage = {
            _class: chunter.class.Message,
            _space: space,
            comments: [comment]
          }
          // absent VDoc fields will be autofilled
          coreService.createVDoc((newMessage as unknown) as VDoc)
        })
      })
    }
  }
</script>

<div class="chat">
  <div><span class="caption-1">Чат {spaceName}</span>&nbsp;</div>
  <ScrollView stylez="height:100%;" autoscroll="{true}">
    <div class="content">
      {#each messages as message (message._id)}
        {#if message.comments}
          <CommentComponent message="{message.comments[0]}" />
        {/if}
      {/each}
    </div>
  </ScrollView>
  <div>
    <ReferenceInput on:message="{(e) => createMessage(e.detail)}" />
  </div>
</div>

<style lang="scss">
  .chat {
    height: 100%;
    display: flex;
    flex-direction: column;

    .content {
      flex-grow: 1;
    }
  }
</style>
