<template>
    <div>
        <h1 class="font-bold text-2xl">New Shipment</h1>

        <div class="bg-white shadow-lg rounded-lg p-5 mt-5">
            <!-- Shipment Items -->
            <h3 class="font-semibold my-4">Shipment Items</h3>

            <div v-for="(_, index) in form.shipmentItems" class="border border-dashed border-gray-300 p-4 rounded-lg mt-4">
                <h3 class="font-semibold uppercase text-xs text-gray-700 mb-3">
                    Item {{ index + 1 }}
                </h3>

                <div class="bg-orange-700 text-white rounded-md p-4 my-4">
                    <b>Weight is valid if these conditions are met:</b>

                    <ul>
                        <li>value > 1</li>
                        <li>someCondition === false (OFF)</li>
                    </ul>
                </div>

                <div class="grid grid-cols-2 gap-x-6 gap-y-4">
                    <div class="flex flex-col gap-2">
                        <div class="flex gap-1">
                            <label class="font-medium uppercase text-xs text-gray-700">Weight</label>
                            <span class="text-red-600 relative -top-1" v-if="r$.$fields.shipmentItems.$each[index].$fields.weight.$isRequired">*</span>
                        </div>
                        
                        <input v-model="form.shipmentItems[index].weight" type="number" class="ring-1 ring-gray-300 rounded outline-none p-2" />
                        <FieldError :errors="r$.$errors.shipmentItems.$each[index].weight" />
                    </div>
                </div>
            </div>

            <button @click="someCondition = !someCondition" class="ml-2 bg-gray-500 text-white px-10 py-3 rounded mt-6 hover:bg-gray-600 transition">
                Toggle someCondition: {{ someCondition ? 'ON' : 'OFF' }}
            </button>
        </div>
    </div>
</template>

<script setup lang="ts">
    import { defineRegleConfig } from '@regle/core'
    import { withMessage } from '@regle/rules'

    const { t } = useI18n()

    const someCondition = ref(true)

    interface Form {
        referenceNumber: string
        shipmentItems: {
            name: string
            quantity: number
            weight: number | string
        }[]
    }

    const form = ref<Form>({
        referenceNumber: '',
        shipmentItems: [
            { name: '', quantity: 0, weight: 0 },
        ]
    })

    const { useRegle } = defineRegleConfig({
        shortcuts: {
            fields: {
                $isRequired: (field) => !!field.$rules.required?.$active
            }
        }
    })

    // Test index & argument
    const extraWeightRule = (myArg: string, index: number) => {
        return withMessage(
            value => {
                return Number(value) > 1 && someCondition.value === false
            },
            t('totally_not_good', { name: myArg + index })
        )
    }
        

    const rules = computed(() => {
        return {
            shipmentItems: {
                $each: (_, index) => ({
                    weight: {
                        extraWeightRule: extraWeightRule('weight banana', index)
                    }
                })
            }
        }
    })

    const { r$ } = useRegle(form, rules, { autoDirty: true  })
</script>
