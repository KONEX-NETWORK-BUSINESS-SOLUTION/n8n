<script setup lang="ts">
import { computed, onMounted, ref, nextTick } from 'vue';
import { useRoute } from 'vue-router';
import WorkflowCanvas from '@/components/canvas/WorkflowCanvas.vue';
import { useNodeTypesStore } from '@/stores/nodeTypes.store';
import { useUIStore } from '@/stores/ui.store';
import { useWorkflowsStore } from '@/stores/workflows.store';
import { useCanvasStore } from '@/stores/canvas.store';
import { useI18n } from '@n8n/i18n';
import { useToast } from '@/composables/useToast';
import type { IWorkflowDb } from '@/Interface';
import { canvasEventBus } from '@/event-bus/canvas';

defineOptions({
	name: 'WorkflowPreview',
});

const route = useRoute();
const i18n = useI18n();
const toast = useToast();

const nodeTypesStore = useNodeTypesStore();
const uiStore = useUIStore();
const workflowsStore = useWorkflowsStore();
const canvasStore = useCanvasStore();

const loading = ref(true);
const workflow = ref<IWorkflowDb | null>(null);

// Computed properties for readonly canvas
const editableWorkflow = computed(() => {
	return workflow.value || { nodes: [], connections: [], pinData: {}, settings: {} };
});

const editableWorkflowObject = computed(() => {
	return workflowsStore.getWorkflowFromUrl(editableWorkflow.value);
});

const workflowId = computed(() => {
	return route.params.id as string;
});

// Load workflow data
const loadWorkflow = async () => {
	try {
		loading.value = true;
		canvasStore.startLoading();
		canvasStore.setLoadingText(i18n.baseText('nodeView.loadingWorkflow'));

		const workflowData = await workflowsStore.fetchWorkflow(workflowId.value);
		workflow.value = workflowData;

		// Set current workflow for stores
		workflowsStore.setWorkflow(workflowData);
		workflowsStore.setActive(true);

		await nextTick();
		canvasStore.stopLoading();
		loading.value = false;

		// Fit view to show all nodes
		setTimeout(() => {
			canvasStore.fitView();
		}, 100);
	} catch (error) {
		console.error('Error loading workflow:', error);
		toast.showError(
			error,
			i18n.baseText('nodeView.couldntLoadWorkflow'),
			i18n.baseText('nodeView.couldntLoadWorkflowMessage'),
		);
		loading.value = false;
		canvasStore.stopLoading();
	}
};

// Handle viewport changes
const onViewportChange = (viewport: any) => {
	canvasStore.setViewport(viewport);
};

// Initialize on mount
onMounted(async () => {
	await loadWorkflow();
});
</script>

<template>
	<div class="workflow-preview">
		<div v-if="loading" class="loading-container">
			<n8n-loading :loading="true" />
		</div>
		<WorkflowCanvas
			v-else
			:workflow-object="editableWorkflowObject"
			:event-bus="canvasEventBus"
			:read-only="true"
			:executing="false"
			:key-bindings="false"
			@viewport:change="onViewportChange"
		/>
	</div>
</template>

<style lang="scss" scoped>
.workflow-preview {
	width: 100%;
	height: 100%;
	position: relative;
}

.loading-container {
	display: flex;
	justify-content: center;
	align-items: center;
	height: 100%;
	width: 100%;
}
</style>
