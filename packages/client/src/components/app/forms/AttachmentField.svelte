<script>
  import Field from "./Field.svelte"
  import { CoreDropzone } from "@budibase/bbui"
  import { FieldType } from "@budibase/types"
  import { getContext } from "svelte"

  export let field
  export let label
  export let disabled = false
  export let readonly = false
  export let compact = false
  export let validation
  export let extensions
  export let onChange
  export let maximum = undefined
  export let span
  export let helpText = null
  export let type = FieldType.ATTACHMENTS
  export let fieldApiMapper = {
    get: value => value,
    set: value => value,
  }
  export let defaultValue = []

  let fieldState
  let fieldApi

  const { API, notificationStore } = getContext("sdk")
  const formContext = getContext("form")
  const BYTES_IN_MB = 1000000

  const handleFileTooLarge = fileSizeLimit => {
    notificationStore.actions.warning(
      `Files cannot exceed ${
        fileSizeLimit / BYTES_IN_MB
      } MB. Please try again with smaller files.`
    )
  }

  const handleTooManyFiles = fileLimit => {
    notificationStore.actions.warning(
      `Please select a maximum of ${fileLimit} files.`
    )
  }

  const processFiles = async fileList => {
    let data = new FormData()
    for (let i = 0; i < fileList.length; i++) {
      data.append("file", fileList[i])
    }
    try {
      return await API.uploadAttachment({
        data,
        tableId: formContext?.dataSource?.tableId,
      })
    } catch (error) {
      return []
    }
  }

  const deleteAttachments = async fileList => {
    try {
      return await API.deleteAttachments({
        keys: fileList,
        tableId: formContext?.dataSource?.tableId,
      })
    } catch (error) {
      return []
    }
  }

  const handleChange = e => {
    const value = fieldApiMapper.set(e.detail)
    const changed = fieldApi.setValue(value)
    if (onChange && changed) {
      onChange({ value })
    }
  }
</script>

<Field
  {label}
  {field}
  {disabled}
  {readonly}
  {validation}
  {span}
  {helpText}
  {type}
  bind:fieldState
  bind:fieldApi
  {defaultValue}
>
  {#if fieldState}
    <CoreDropzone
      value={fieldApiMapper.get(fieldState.value)}
      disabled={fieldState.disabled || fieldState.readonly}
      error={fieldState.error}
      on:change={handleChange}
      {processFiles}
      {deleteAttachments}
      {handleFileTooLarge}
      {handleTooManyFiles}
      {maximum}
      {extensions}
      {compact}
    />
  {/if}
</Field>
