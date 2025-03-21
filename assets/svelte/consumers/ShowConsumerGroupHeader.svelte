<script lang="ts">
  import { onMount } from "svelte";
  import {
    ArrowLeft,
    Clock,
    RotateCw,
    Radio,
    AlertCircle,
  } from "lucide-svelte";
  import { Button } from "$lib/components/ui/button";
  import * as Dialog from "$lib/components/ui/dialog";
  import { formatRelativeTimestamp } from "../utils";
  import LinkPushNavigate from "$lib/components/LinkPushNavigate.svelte";

  export let consumer;
  export let live_action;
  export let live;
  export let parent;
  export let messages_failing;

  let showDeleteConfirmDialog = false;
  let deleteConfirmDialogLoading = false;

  function handleEdit() {
    live.pushEventTo("#" + parent, "edit", {});
  }

  function handleDelete() {
    showDeleteConfirmDialog = true;
  }

  function cancelDelete() {
    showDeleteConfirmDialog = false;
  }

  function confirmDelete() {
    deleteConfirmDialogLoading = true;
    live.pushEventTo("#" + parent, "delete", {}, () => {
      showDeleteConfirmDialog = false;
      deleteConfirmDialogLoading = false;
    });
  }

  let activeTab: string;

  $: messageUrl = messages_failing
    ? `${consumer.href}/messages?showAcked=false`
    : `${consumer.href}/messages`;

  onMount(() => {
    activeTab = live_action === "messages" ? "messages" : "overview";
  });
</script>

<div class="bg-white border-b header">
  <div class="container mx-auto px-4 py-4">
    <div class="flex items-center justify-between">
      <div class="flex items-center space-x-4">
        <LinkPushNavigate href="/consumer-groups">
          <Button variant="ghost" size="sm">
            <ArrowLeft class="h-4 w-4" />
          </Button>
        </LinkPushNavigate>
        <div class="flex items-center">
          <Radio class="h-6 w-6 mr-2" />
          <h1 class="text-xl font-semibold">Consumer Group: {consumer.name}</h1>
        </div>
      </div>
      <div class="flex items-center space-x-4">
        <div
          class="hidden lg:flex flex-col items-left gap-1 text-xs text-gray-500"
        >
          <div class="flex items-center gap-2">
            <Clock class="h-4 w-4" />
            <span>Created {formatRelativeTimestamp(consumer.inserted_at)}</span>
          </div>
          <div class="flex items-center gap-2">
            <RotateCw class="h-4 w-4" />
            <span>Updated {formatRelativeTimestamp(consumer.updated_at)}</span>
          </div>
        </div>
        <Button variant="outline" size="sm" on:click={handleEdit}>Edit</Button>
        <Button
          variant="outline"
          size="sm"
          class="text-red-600 hover:text-red-700"
          on:click={handleDelete}
        >
          Delete
        </Button>
      </div>
    </div>
  </div>

  <div class="container mx-auto px-4">
    <div class="flex space-x-4">
      <a
        href={consumer.href}
        class={`py-2 px-4 font-medium border-b-2 ${
          activeTab === "overview"
            ? "text-black border-black"
            : "text-gray-500 hover:text-gray-700 border-transparent"
        }`}
        data-phx-link="redirect"
        data-phx-link-state="push"
      >
        Overview
      </a>
      <a
        href={messageUrl}
        class={`py-2 px-4 flex items-center font-medium border-b-2 ${
          activeTab === "messages"
            ? "text-black border-black"
            : "text-gray-500 hover:text-gray-700 border-transparent"
        }`}
        data-phx-link="redirect"
        data-phx-link-state="push"
      >
        Messages
        {#if messages_failing}
          <AlertCircle class="h-4 w-4 text-red-600 ml-1" />
        {/if}
      </a>
    </div>
  </div>
</div>

<Dialog.Root bind:open={showDeleteConfirmDialog}>
  <Dialog.Content>
    <Dialog.Header>
      <Dialog.Title class="leading-6">
        Are you sure you want to delete this Consumer Group?
      </Dialog.Title>
      <Dialog.Description>This action cannot be undone.</Dialog.Description>
    </Dialog.Header>
    <Dialog.Footer>
      <Button variant="outline" on:click={cancelDelete}>Cancel</Button>
      <Button
        variant="destructive"
        on:click={confirmDelete}
        disabled={deleteConfirmDialogLoading}
      >
        {#if deleteConfirmDialogLoading}
          Deleting...
        {:else}
          Delete
        {/if}
      </Button>
    </Dialog.Footer>
  </Dialog.Content>
</Dialog.Root>
