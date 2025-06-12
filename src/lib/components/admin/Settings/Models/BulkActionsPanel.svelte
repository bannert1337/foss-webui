<script lang="ts">
	import { getContext } from 'svelte';
	import { getModels, toggleModelsById, updateModelsById } from '$lib/apis/models';
	import { models, config, settings, user } from '$lib/stores';
	import { toast } from 'svelte-sonner';

	export let selectedModelIds: string[] = [];
	export let onActionComplete: () => void;

	type Tag = { name: string };
	type Visibility = 'public' | 'private' | 'shared';
	type AccessControl = { visibility: Visibility };
	type Capabilities = { vision?: boolean; usage?: boolean; citations?: boolean };
	type ModelUpdate = {
		tags?: Tag[];
		tag_action?: 'add' | 'remove';
		is_hidden?: boolean;
		access_control?: AccessControl;
		meta?: { capabilities?: Capabilities };
	};

	let tagInput: string = '';
	let selectedVisibility: Visibility = 'public';
	let visionEnabled: boolean = false;
	let usageEnabled: boolean = false;
	let citationsEnabled: boolean = false;

	const handleActivate = async () => {
		const token = localStorage.token;
		try {
			await toggleModelsById(token, selectedModelIds, true);
			toast.success('Models activated successfully!');
			models.set(await getModels(token));
			onActionComplete();
		} catch (error) {
			toast.error('Failed to activate models.');
			console.error('Error activating models:', error);
		}
	};

	const handleDeactivate = async () => {
		const token = localStorage.token;
		try {
			await toggleModelsById(token, selectedModelIds, false);
			toast.success('Models deactivated successfully!');
			models.set(await getModels(token));
			onActionComplete();
		} catch (error) {
			toast.error('Failed to deactivate models.');
			console.error('Error deactivating models:', error);
		}
	};

	const handleHide = async () => {
		const token = localStorage.token;
		try {
			await updateModelsById(token, selectedModelIds, { is_hidden: true } as ModelUpdate);
			toast.success('Models hidden successfully!');
			models.set(await getModels(token));
			onActionComplete();
		} catch (error) {
			toast.error('Failed to hide models.');
			console.error('Error hiding models:', error);
		}
	};

	const handleShow = async () => {
		const token = localStorage.token;
		try {
			await updateModelsById(token, selectedModelIds, { is_hidden: false } as ModelUpdate);
			toast.success('Models shown successfully!');
			models.set(await getModels(token));
			onActionComplete();
		} catch (error) {
			toast.error('Failed to show models.');
			console.error('Error showing models:', error);
		}
	};

	const handleAddTags = async () => {
		const token = localStorage.token;
		const tags = tagInput
			.split(',')
			.map((tag) => ({ name: tag.trim() }))
			.filter((tag) => tag.name);
		if (tags.length === 0) {
			toast.info('Please enter tags to add.');
			return;
		}
		try {
			await updateModelsById(token, selectedModelIds, { tags, tag_action: 'add' } as ModelUpdate);
			toast.success('Tags added successfully!');
			models.set(await getModels(token));
			onActionComplete();
			tagInput = '';
		} catch (error) {
			toast.error('Failed to add tags.');
			console.error('Error adding tags:', error);
		}
	};

	const handleRemoveTags = async () => {
		const token = localStorage.token;
		const tags = tagInput
			.split(',')
			.map((tag) => ({ name: tag.trim() }))
			.filter((tag) => tag.name);
		if (tags.length === 0) {
			toast.info('Please enter tags to remove.');
			return;
		}
		try {
			await updateModelsById(token, selectedModelIds, {
				tags,
				tag_action: 'remove'
			} as ModelUpdate);
			toast.success('Tags removed successfully!');
			models.set(await getModels(token));
			onActionComplete();
			tagInput = '';
		} catch (error) {
			toast.error('Failed to remove tags.');
			console.error('Error removing tags:', error);
		}
	};

	const handleVisibilityChange = async () => {
		const token = localStorage.token;
		try {
			await updateModelsById(token, selectedModelIds, {
				access_control: { visibility: selectedVisibility }
			} as ModelUpdate);
			toast.success('Visibility updated successfully!');
			models.set(await getModels(token));
			onActionComplete();
		} catch (error) {
			toast.error('Failed to update visibility.');
			console.error('Error updating visibility:', error);
		}
	};

	const handleCapabilitiesChange = async () => {
		const token = localStorage.token;
		try {
			await updateModelsById(token, selectedModelIds, {
				meta: {
					capabilities: {
						vision: visionEnabled,
						usage: usageEnabled,
						citations: citationsEnabled
					}
				}
			} as ModelUpdate);
			toast.success('Capabilities updated successfully!');
			models.set(await getModels(token));
			onActionComplete();
		} catch (error) {
			toast.error('Failed to update capabilities.');
			console.error('Error updating capabilities:', error);
		}
	};
</script>

<div class="bulk-actions-panel">
	<h3>Bulk Actions</h3>
	<div class="actions">
		<button on:click={handleActivate}>Activate</button>
		<button on:click={handleDeactivate}>Deactivate</button>
		<button on:click={handleHide}>Hide</button>
		<button on:click={handleShow}>Show</button>
	</div>
</div>

<style>
	.bulk-actions-panel {
		padding: 1rem;
		border: 1px solid #ccc;
		border-radius: 8px;
		background-color: #f9f9f9;
		margin-top: 1rem;
	}

	.bulk-actions-panel h3 {
		margin-top: 0;
		margin-bottom: 1rem;
	}

	.bulk-actions-panel .actions button {
		margin-right: 0.5rem;
		padding: 0.5rem 1rem;
		border: 1px solid #007bff;
		border-radius: 4px;
		background-color: #007bff;
		color: white;
		cursor: pointer;
	}

	.bulk-actions-panel .actions button:hover {
		background-color: #0056b3;
	}
</style>
