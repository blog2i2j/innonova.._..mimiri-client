<template>
	<div class="flex flex-col h-full">
		<TabBar :items="['Current Plan']"></TabBar>
		<div class="flex flex-col overflow-y-auto">
			<div class="p-1 pt-2 flex" :data-testid="populated ? 'home-view' : ''">
				<SubscriptionItem
					v-if="product"
					:product="product"
					:subscription="subscription"
					:showFeatures="true"
					:showResume="subscription?.renewalType === 'none'"
					:showUpgrade="product.sku === 'free' && subscription?.renewalType !== 'none'"
					:showChange="product.sku !== 'free' && subscription?.renewalType !== 'none'"
					:showCancel="product.sku !== 'free' && subscription?.renewalType !== 'none'"
					:currency="subscription?.renewalCurrency"
					:showStatus="true"
					@change="change"
					@cancel="cancel"
					@resume="resume"
					@pay-invoice="payInvoice"
				></SubscriptionItem>
				<input type="hidden" data-testid="current-subscription-sku" :value="product?.sku" />
				<input type="hidden" data-testid="current-subscription-paid-until" :value="subscription?.paidUntil" />
			</div>
			<div class="p-1 mt-4">
				Read more about plans <a href="https://mimiri.io/subscription" target="_blank">here</a>
			</div>
			<div class="p-1 mt-4 leading-6 mb-10">
				<ItemHeader>Beta information</ItemHeader>
				<p>We are currently working to perfect the subscription processes.</p>
				<p>If you encounter any issues please don't hesitate to contact us:</p>
				<ul class="mt-1">
					<li><a href="https://discord.gg/pg69qPAVZR" target="_blank">Discord</a></li>
					<li><a href="https://www.reddit.com/r/mimiri/" target="_blank">Reddit</a></li>
					<li class="flex gap-2">
						info@innonova.ch<CopyIcon
							v-if="!copied"
							title="copy"
							@click="copyEmail"
							class="w-5 hover:w-6 cursor-pointer"
						></CopyIcon>
						<div v-if="copied" class="ml-1 cursor-default select-none">Copied</div>
					</li>
				</ul>
			</div>
		</div>
	</div>
</template>

<script setup lang="ts">
import { clipboardManager, noteManager } from '../../global'
import { type Invoice, type Subscription, type SubscriptionProduct } from '../../services/types/subscription'
import ItemHeader from './ItemHeader.vue'
import CopyIcon from '../../icons/copy.vue'
import SubscriptionItem from './SubscriptionItem.vue'
import { onMounted, ref } from 'vue'
import TabBar from '../elements/TabBar.vue'

const product = ref<SubscriptionProduct>()
const subscription = ref<Subscription>()
const populated = ref(false)
const copied = ref(false)

const emit = defineEmits(['change', 'pay-invoice'])

onMounted(async () => {
	await populate()
})

const populate = async () => {
	populated.value = false
	product.value = await noteManager.paymentClient.getCurrentSubscriptionProduct()
	subscription.value = await noteManager.paymentClient.getCurrentSubscription()
	populated.value = true
}

const change = async () => {
	emit('change')
}

const cancel = async () => {
	await noteManager.paymentClient.cancelSubscription()
	await populate()
}

const resume = async () => {
	await noteManager.paymentClient.resumeSubscription()
	await populate()
}

const payInvoice = (invoice: Invoice) => {
	emit('pay-invoice', invoice)
}

const copyEmail = () => {
	clipboardManager.write('info@innonova.ch')
	copied.value = true
	setTimeout(() => (copied.value = false), 1000)
}

defineExpose({
	populate,
})
</script>
