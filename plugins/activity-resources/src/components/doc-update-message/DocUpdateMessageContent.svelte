<!--
// Copyright © 2023 Hardcore Engineering Inc.
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
-->
<script lang="ts">
  import { AttachedDoc, Attribute, Class, Collection, Doc, Ref } from '@hcengineering/core'
  import { Icon, Label } from '@hcengineering/ui'
  import { IntlString } from '@hcengineering/platform'
  import { getClient } from '@hcengineering/presentation'
  import view from '@hcengineering/view'
  import activity, { DisplayDocUpdateMessage, DocUpdateMessage, DocUpdateMessageViewlet } from '@hcengineering/activity'

  import DocUpdateMessageObjectValue from './DocUpdateMessageObjectValue.svelte'

  export let message: DisplayDocUpdateMessage
  export let viewlet: DocUpdateMessageViewlet | undefined
  export let objectName: IntlString
  export let collectionName: IntlString | undefined
  export let objectClass: Ref<Class<Doc>>
  export let collectionAttribute: Attribute<Collection<AttachedDoc>> | undefined = undefined

  const client = getClient()
  const hierarchy = client.getHierarchy()
  const clazz = hierarchy.getClass(message.objectClass)

  const objectPanel = hierarchy.classHierarchyMixin(objectClass, view.mixin.ObjectPanel)
  const objectPresenter = hierarchy.classHierarchyMixin(objectClass, view.mixin.ObjectPresenter)

  const isOwn = message.objectId === message.attachedTo

  let valueMessages: DocUpdateMessage[] = []

  $: valueMessages = message.previousMessages?.length ? [...message.previousMessages, message] : [message]
  $: hasDifferentActions = message.previousMessages?.some(({ action }) => action !== message.action)
  $: icon = viewlet?.icon ?? collectionAttribute?.icon ?? clazz.icon ?? activity.icon.Activity
</script>

<div class="content">
  <span class="mr-1">
    <Icon {icon} size="small" />
  </span>
  {#if hasDifferentActions}
    <Label label={activity.string.UpdatedCollection} />
  {:else if message.action === 'create'}
    <Label label={activity.string.New} />
  {:else if message.action === 'remove' && message.updateCollection}
    <Label label={activity.string.Removed} />
  {/if}
  <span class="lower">
    {#if collectionName && (message.previousMessages?.length || !isOwn)}
      <Label label={collectionName} />:
    {:else}
      <Label label={objectName} />:
    {/if}
  </span>

  {#if hasDifferentActions}
    {@const removeMessages = valueMessages.filter(({ action }) => action === 'remove')}
    {@const createMessages = valueMessages.filter(({ action }) => action === 'create')}

    {#each createMessages as valueMessage, index}
      <DocUpdateMessageObjectValue
        message={valueMessage}
        {objectPresenter}
        {objectPanel}
        {viewlet}
        withIcon={index === 0}
        hasSeparator={createMessages.length > 1 && index !== createMessages.length - 1}
      />
    {/each}
    {#each removeMessages as valueMessage, index}
      <DocUpdateMessageObjectValue
        message={valueMessage}
        {objectPresenter}
        {objectPanel}
        {viewlet}
        withIcon={index === 0}
        hasSeparator={removeMessages.length > 1 && index !== removeMessages.length - 1}
      />
    {/each}
  {:else}
    {#each valueMessages as valueMessage, index}
      <DocUpdateMessageObjectValue
        message={valueMessage}
        {objectPresenter}
        {objectPanel}
        {viewlet}
        hasSeparator={valueMessages.length > 1 && index !== valueMessages.length - 1}
      />
    {/each}
  {/if}
</div>

<style lang="scss">
  .content {
    display: flex;
    gap: 0.25rem;
    align-items: center;
    flex-wrap: wrap;
  }
</style>
