<template>
	<div
		ref="toolbar"
		class="flex items-center py-px px-1.5 bg-toolbar border-b border-solid border-toolbar mobile:justify-between"
		data-testid="main-toolbar"
	>
		<ToolbarIcon v-if="!mimiriPlatform.isDesktop" icon="menu" @click="showMobileMenu"></ToolbarIcon>
		<div
			v-if="!mimiriPlatform.isDesktop"
			class="inline-block h-4/5 w-0 border border-solid border-toolbar-separator m-0.5"
		></div>
		<ToolbarIcon icon="plus-small" :hoverEffect="true" title="New Root Note" @click="showCreateMenu"></ToolbarIcon>
		<ToolbarIcon
			:icon="settingsManager.lastNoteCreateType === 'child' ? 'add-note' : 'add-sibling-note'"
			:disabled="
				!noteManager.selectedNote || noteManager.selectedNote.isSystem || noteManager.selectedNote.isInRecycleBin
			"
			:hoverEffect="true"
			:title="settingsManager.lastNoteCreateType === 'child' ? 'New Child Note' : 'New Sibling Note'"
			@click="createChildNote"
		></ToolbarIcon>
		<div class="inline-block h-4/5 w-0 border border-solid border-toolbar-separator m-0.5"></div>
		<ToolbarIcon
			class="desktop:hidden"
			icon="search-all-notes"
			:hoverEffect="true"
			title="Search All Notes"
			@click="toggleSearchAllNotes"
		></ToolbarIcon>
		<ToolbarIcon
			class="hidden! desktop:block!"
			icon="search-all-notes"
			:hoverEffect="true"
			title="Search All Notes"
			@click="gotoSearchAllNotes"
		></ToolbarIcon>
		<div
			v-if="mimiriPlatform.isPhone"
			class="inline-block h-4/5 w-0 border border-solid border-toolbar-separator m-0.5"
		></div>
		<ToolbarIcon
			v-if="mimiriPlatform.isPhone"
			:icon="notificationManager.unread > 0 ? 'notifications-active' : 'notifications'"
			:class="{ 'mt-0.5': notificationManager.unread === 0 }"
			:hoverEffect="true"
			:disabled="notificationManager.count === 0"
			title="Notifications"
			@click="notificationsClick"
		></ToolbarIcon>
		<ToolbarIcon
			v-if="mimiriPlatform.isPhone"
			:icon="accountIcon"
			:hoverEffect="true"
			title="Account"
			@click="accountClick"
		></ToolbarIcon>
	</div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import { noteManager, showSearchBox, ipcClient, searchInput } from '../global'
import ToolbarIcon from './ToolbarIcon.vue'
import { MenuItems, menuManager } from '../services/menu-manager'
import { mimiriPlatform } from '../services/mimiri-platform'
import { notificationManager } from '../global'
import { settingsManager } from '../services/settings-manager'

const toolbar = ref(null)
const accountIcon = computed(() => {
	if (!ipcClient.isAvailable) {
		return 'account'
	}
	if (noteManager.state.online && noteManager.state.authenticated) {
		return 'account-online'
	}
	return 'account-offline'
})

const notificationsClick = () => {
	console.log(notificationManager.count)

	if (notificationManager.count > 0) {
		notificationManager.show()
	}
}
const accountClick = () => {
	const rect = toolbar.value.getBoundingClientRect()
	menuManager.showMenu({ x: screen.width, y: rect.bottom, alignRight: true }, [
		MenuItems.ChangeUsername,
		MenuItems.ChangePassword,
		MenuItems.DeleteAccount,
		MenuItems.Separator,
		...(noteManager.isAnonymous ? [MenuItems.CreatePassword, MenuItems.Login] : [MenuItems.Logout]),
	])
}

const createChildNote = () => {
	if (settingsManager.lastNoteCreateType === 'sibling') {
		if (noteManager.selectedNote.parent.isRoot) {
			noteManager.newRootNote()
			return
		}
		noteManager.selectedNote.parent.select()
	}
	noteManager.newNote()
}

const searchAllNotes = () => {
	showSearchBox.value = true
}

const toggleSearchAllNotes = () => {
	showSearchBox.value = !showSearchBox.value
}

const gotoSearchAllNotes = () => {
	searchInput.value.focus()
	searchInput.value.classList.add('animate-ping')
	setTimeout(() => {
		searchInput.value.classList.remove('animate-ping')
	}, 600)
}

const showCreateMenu = () => {
	const createMenu = [MenuItems.NewRootNote, MenuItems.NewChildNote, MenuItems.NewSiblingNote]

	const rect = toolbar.value.getBoundingClientRect()

	menuManager.showMenu({ x: 10, y: rect.bottom }, noteManager.selectedNote ? createMenu : createMenu)
}

const showMobileMenu = () => {
	const isSystem = !!noteManager.selectedNote?.isSystem
	const isRecycleBin = !!noteManager.selectedNote?.isRecycleBin
	const isInRecycleBin = !!noteManager.selectedNote?.isInRecycleBin
	const isShared = !!noteManager.selectedNote?.isShared
	const isShareRoot = !!noteManager.selectedNote?.isShareRoot
	const whenSelectedNote = [
		MenuItems.About,
		MenuItems.DarkMode,
		...(isRecycleBin ? [MenuItems.Separator, MenuItems.EmptyRecycleBin] : []),
		...(!isSystem && !isInRecycleBin
			? [
					MenuItems.Separator,
					MenuItems.NewNote,
					MenuItems.NewRootNote,
					MenuItems.Separator,
					MenuItems.Duplicate,
					MenuItems.Cut,
					MenuItems.Copy,
					MenuItems.Paste,
				]
			: []),

		...(!isSystem && isInRecycleBin ? [MenuItems.Separator, MenuItems.Cut, MenuItems.Copy] : []),
		MenuItems.Separator,
		...(!isSystem && !isInRecycleBin && (!isShared || isShareRoot) ? [MenuItems.Share] : []),
		...(!isSystem && !isInRecycleBin && !isShared ? [MenuItems.ReceiveShare] : []),
		MenuItems.Refresh,
		...(!isSystem && !isInRecycleBin ? [MenuItems.Separator, MenuItems.Rename, MenuItems.Recycle] : []),
		...(!isSystem && isInRecycleBin ? [MenuItems.Separator, MenuItems.Delete] : []),
		...(!isSystem ? [MenuItems.Separator, MenuItems.Properties] : []),
	]
	const whenNoSelectedNote = [MenuItems.About, MenuItems.DarkMode]

	const rect = toolbar.value.getBoundingClientRect()

	menuManager.showMenu({ x: 10, y: rect.bottom }, noteManager.selectedNote ? whenSelectedNote : whenNoSelectedNote)
}

defineExpose({
	searchAllNotes,
})
</script>
