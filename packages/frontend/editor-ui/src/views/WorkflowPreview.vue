<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { useRoute } from 'vue-router';
import { useI18n } from '@n8n/i18n';
import { useToast } from '@/composables/useToast';

defineOptions({
	name: 'WorkflowPreview',
});

const route = useRoute();
const i18n = useI18n();
const toast = useToast();

// Component state
const workflow = ref<any>(null);
const isLoading = ref(true);
const error = ref<string | null>(null);

const workflowId = route.params.id as string;

// Load workflow data
const loadWorkflow = async () => {
	try {
		isLoading.value = true;
		error.value = null;

		const response = await fetch(`/rest/workflows/preview/${workflowId}`);

		if (!response.ok) {
			throw new Error(`HTTP error! status: ${response.status}`);
		}

		const data = await response.json();

		// Ensure workflow data has proper structure
		workflow.value = {
			id: data.data?.id || data.id,
			name: data.data?.name || data.name,
			nodes: data.data?.nodes || data.nodes || [],
			connections: data.data?.connections || data.connections || {},
			settings: data.data?.settings || data.settings || {},
			pinData: data.data?.pinData || data.pinData || {},
		};

		console.log('Loaded workflow:', workflow.value);
	} catch (err) {
		console.error('Error loading workflow:', err);
		error.value = 'Failed to load workflow';
		toast.showError(
			i18n.baseText('workflowPreview.errorLoadingWorkflow'),
			i18n.baseText('workflowPreview.errorLoadingWorkflowMessage'),
		);
	} finally {
		isLoading.value = false;
	}
};

// Load workflow on mount
onMounted(async () => {
	await loadWorkflow();
});
</script>

<template>
	<div class="workflow-preview">
		<div v-if="isLoading" class="loading">
			<div class="loading-spinner"></div>
			<p>{{ i18n.baseText('workflowPreview.loading') }}</p>
		</div>

		<div v-else-if="error" class="error-message">
			<p>{{ error }}</p>
			<button @click="loadWorkflow">Retry</button>
		</div>

		<div v-else-if="workflow" class="workflow-content">
			<h2>{{ workflow.name }}</h2>
			<div class="workflow-info">
				<p><strong>ID:</strong> {{ workflow.id }}</p>
				<p><strong>Nodes:</strong> {{ workflow.nodes.length }}</p>
				<p><strong>Connections:</strong> {{ Object.keys(workflow.connections).length }}</p>
			</div>

			<!-- Simple workflow visualization -->
			<div class="workflow-nodes">
				<h3>Nodes:</h3>
				<div v-for="node in workflow.nodes" :key="node.id" class="node-item">
					<div class="node-header">
						<strong>{{ node.name }}</strong>
						<span class="node-type">({{ node.type }})</span>
					</div>
					<div class="node-position">
						Position: [{{ node.position?.[0] || 0 }}, {{ node.position?.[1] || 0 }}]
					</div>
				</div>
			</div>

			<!-- Connections -->
			<div v-if="Object.keys(workflow.connections).length > 0" class="workflow-connections">
				<h3>Connections:</h3>
				<pre>{{ JSON.stringify(workflow.connections, null, 2) }}</pre>
			</div>
		</div>

		<div v-else class="error-message">
			<p>{{ i18n.baseText('workflowPreview.workflowNotFound') }}</p>
		</div>
	</div>
</template>

<style lang="scss" scoped>
.workflow-preview {
	height: 100vh;
	width: 100vw;
	padding: 20px;
	background: var(--color-background-light);
	overflow: auto;
}

.loading {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	height: 100%;

	.loading-spinner {
		width: 40px;
		height: 40px;
		border: 3px solid var(--color-foreground-light);
		border-top: 3px solid var(--color-primary);
		border-radius: 50%;
		animation: spin 1s linear infinite;
	}

	p {
		margin-top: 16px;
		color: var(--color-text-base);
	}
}

.error-message {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	height: 100%;

	p {
		color: var(--color-danger);
		font-size: 16px;
		margin-bottom: 16px;
	}

	button {
		padding: 8px 16px;
		background: var(--color-primary);
		color: white;
		border: none;
		border-radius: 4px;
		cursor: pointer;

		&:hover {
			background: var(--color-primary-shade-1);
		}
	}
}

.workflow-content {
	max-width: 1200px;
	margin: 0 auto;

	h2 {
		color: var(--color-text-dark);
		margin-bottom: 20px;
		font-size: 24px;
	}

	h3 {
		color: var(--color-text-base);
		margin: 20px 0 10px 0;
		font-size: 18px;
	}
}

.workflow-info {
	background: var(--color-background-base);
	padding: 16px;
	border-radius: 8px;
	margin-bottom: 20px;

	p {
		margin: 4px 0;
		color: var(--color-text-base);
	}
}

.workflow-nodes {
	background: var(--color-background-base);
	padding: 16px;
	border-radius: 8px;
	margin-bottom: 20px;
}

.node-item {
	background: var(--color-background-light);
	padding: 12px;
	margin: 8px 0;
	border-radius: 6px;
	border: 1px solid var(--color-foreground-light);

	.node-header {
		display: flex;
		align-items: center;
		gap: 8px;
		margin-bottom: 4px;

		strong {
			color: var(--color-text-dark);
		}

		.node-type {
			color: var(--color-text-light);
			font-size: 12px;
		}
	}

	.node-position {
		color: var(--color-text-base);
		font-size: 12px;
	}
}

.workflow-connections {
	background: var(--color-background-base);
	padding: 16px;
	border-radius: 8px;

	pre {
		background: var(--color-background-light);
		padding: 12px;
		border-radius: 4px;
		overflow-x: auto;
		font-size: 12px;
		color: var(--color-text-base);
	}
}

@keyframes spin {
	0% {
		transform: rotate(0deg);
	}
	100% {
		transform: rotate(360deg);
	}
}
</style>
