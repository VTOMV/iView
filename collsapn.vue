<template>
    <div :class="classes" @input="$emit('input',$event.value)">
        <slot></slot>
    </div>
</template>
<script>
    const prefixCls = 'ivu-collapse';

    export default {
        name: 'Collapse',
        model: {
            prop: "value",
            event: "input"
        },
        props: {
            value: {
                type: Boolean,
                default: false
            },
            accordion: {
                type: Boolean,
                default: false
            },
            data: {
                type: [Array, String]
            },
            simple: {
                type: Boolean,
                default: false
            }
        },
        data() {
            return {
                currentValue: this.data
            };
        },
        computed: {
            isActive: {
                get() {
                    return this.value;
                },
                set(val) {
                    this.$emit("input", val)
                }
            },
            classes() {
                return [
                    `${prefixCls}`,
                    {
                        [`${prefixCls}-simple`]: this.simple
                    }
                ];
            }
        },
        mounted() {
            this.setActive();
        },
        methods: {
            setActive() {
                const activeKey = this.getActiveKey();

                this.$children.forEach((child, index) => {
                    const name = child.name || index.toString();
                    child.isActive = activeKey.indexOf(name) > -1;
                    child.index = index;
                });
            },
            getActiveKey() {
                let activeKey = this.currentValue || [];
                const accordion = this.accordion;

                if (!Array.isArray(activeKey)) {
                    activeKey = [activeKey];
                }

                if (accordion && activeKey.length > 1) {
                    activeKey = [activeKey[0]];
                }

                for (let i = 0; i < activeKey.length; i++) {
                    activeKey[i] = activeKey[i].toString();
                }

                return activeKey;
            },
            toggle(data) {
                const name = data.name.toString();
                let newActiveKey = [];

                if (this.accordion) {
                    if (!data.isActive) {
                        newActiveKey.push(name);
                    }
                } else {
                    let activeKey = this.getActiveKey();
                    const nameIndex = activeKey.indexOf(name);

                    if (data.isActive) {
                        if (nameIndex > -1) {
                            activeKey.splice(nameIndex, 1);
                        }
                    } else {
                        if (nameIndex < 0) {
                            activeKey.push(name);
                        }
                    }

                    newActiveKey = activeKey;
                }
                this.isActive = data.isActive;
                this.currentValue = newActiveKey;
                this.$emit('input', newActiveKey);
                this.$emit('on-change', newActiveKey);
            },
            objIsEmpty(obj) {
                let objectType = this.getType(obj);
                if (objectType === "null") {
                    return true;
                } else if (objectType === "undefined") {
                    return true;
                } else if (objectType === "Object") {
                    if (Object.keys(obj).length == 0) {
                        return true;
                    } else {
                        return false;
                    }
                } else if (objectType === "Array") {
                    console.log(obj);
                    if (obj.length == 0) {
                        return true;
                    } else {
                        return false;
                    }
                } else if (objectType == "string") {
                    if (obj == "") {
                        return true;
                    } else {
                        return false;
                    }
                } else {
                    return false;
                }
            },
            getType(obj) {
                let type = typeof obj;
                if (type != "object") {
                    return type;
                }
                return Object.prototype.toString.call(obj).replace(/^\[object (\S+)\]$/, '$1');
            }
        },
        watch: {
            data(val) {
                this.currentValue = val;
            },
            currentValue() {
                this.setActive();
            }
        }
    };
</script>
