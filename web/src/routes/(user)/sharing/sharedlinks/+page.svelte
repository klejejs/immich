<script lang="ts">
	import ControlAppBar from '$lib/components/shared-components/control-app-bar.svelte';
	import ArrowLeft from 'svelte-material-icons/ArrowLeft.svelte';
	import { api, SharedLinkResponseDto } from '@api';
	import { goto } from '$app/navigation';
	import SharedLinkCard from '$lib/components/sharedlinks-page/shared-link-card.svelte';
	import {
		notificationController,
		NotificationType
	} from '$lib/components/shared-components/notification/notification';
	import { onMount } from 'svelte';
	import CreateSharedLinkModal from '$lib/components/shared-components/create-share-link-modal/create-shared-link-modal.svelte';
	import ConfirmDialogue from '$lib/components/shared-components/confirm-dialogue.svelte';
	import { handleError } from '$lib/utils/handle-error';
	import { AppRoute } from '$lib/constants';

	let sharedLinks: SharedLinkResponseDto[] = [];
	let editSharedLink: SharedLinkResponseDto | null = null;

	let deleteLinkId: string | null = null;

	const refresh = async () => {
		const { data } = await api.sharedLinkApi.getAllSharedLinks();
		sharedLinks = data;
	};

	onMount(async () => {
		await refresh();
	});

	const handleDeleteLink = async () => {
		if (!deleteLinkId) {
			return;
		}

		try {
			await api.sharedLinkApi.removeSharedLink({ id: deleteLinkId });
			notificationController.show({ message: 'Deleted shared link', type: NotificationType.Info });
			deleteLinkId = null;
			refresh();
		} catch (error) {
			handleError(error, 'Unable to delete shared link');
		}
	};

	const handleEditDone = async () => {
		refresh();
		editSharedLink = null;
	};

	const handleCopyLink = async (key: string) => {
		const link = `${window.location.origin}/share/${key}`;
		await navigator.clipboard.writeText(link);
		notificationController.show({
			message: 'Link copied to clipboard',
			type: NotificationType.Info
		});
	};
</script>

<ControlAppBar backIcon={ArrowLeft} on:close-button-click={() => goto(AppRoute.SHARING)}>
	<svelte:fragment slot="leading">Shared links</svelte:fragment>
</ControlAppBar>

<section class="flex flex-col pb-[120px] mt-[120px]">
	<div class="w-[50%] m-auto mb-4 dark:text-immich-gray">
		<p>Manage shared links</p>
	</div>
	{#if sharedLinks.length === 0}
		<div
			class="w-[50%] m-auto bg-gray-100 flex place-items-center place-content-center rounded-lg p-12"
		>
			<p>You don't have any shared links</p>
		</div>
	{:else}
		<div class="flex flex-col w-[50%] m-auto">
			{#each sharedLinks as link (link.id)}
				<SharedLinkCard
					{link}
					on:delete={() => (deleteLinkId = link.id)}
					on:edit={() => (editSharedLink = link)}
					on:copy={() => handleCopyLink(link.key)}
				/>
			{/each}
		</div>
	{/if}
</section>

{#if editSharedLink}
	<CreateSharedLinkModal
		editingLink={editSharedLink}
		shareType={editSharedLink.type}
		album={editSharedLink.album}
		on:close={handleEditDone}
	/>
{/if}

{#if deleteLinkId}
	<ConfirmDialogue
		title="Delete Shared Link"
		prompt="Are you want to delete this shared link?"
		confirmText="Delete"
		on:confirm={() => handleDeleteLink()}
		on:cancel={() => (deleteLinkId = null)}
	/>
{/if}
