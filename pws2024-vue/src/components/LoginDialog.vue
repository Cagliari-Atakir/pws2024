<script>

    const authEndpoint = '/api/auth'

    export default {
        data() {
            return {
                isValid: false,
                input: { username: '', password: '' },
                rules: {
                    required: value => {
                        return value.length > 0 || 'Required field'
                    }
                }
            }
        },
        emits: [ 'close' ],
        methods: {
           send() {
              fetch(authEndpoint, {
                method: 'POST',
                headers: { 'Content-type': 'application/json' },
                body: JSON.stringify(this.input)
              }).then(res => {
                res.json().then(data => {
                    if(!res.ok) {
                        this.$emit('close', data.error, 'error')
                    } else {
                        this.input = {}
                        this.$emit('close', 'Successful login')
                    }
                }).catch(err => {
                    console.error(err)
                    this.$emit('close', 'Backend does not respond', 'error')
                })
              })
           },
           close() {
                this.$emit('close')
           }  
        }
    }
</script>

<template>
    <v-form v-model="isValid">
        <v-card>
            <v-card-title>Login</v-card-title>
            <v-card-text>
                <v-text-field variant="outlined" label="User name" v-model="input.username" :rules="[ rules.required ]">
                </v-text-field>
                <v-text-field variant="outlined" type="password" label="Password" v-model="input.password" :rules="[]">
                </v-text-field>
            </v-card-text>
            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="primary" variant="elevated" @click="send" :disabled="!isValid">Login</v-btn>
                <v-btn variant="elevated" @click="close">Close</v-btn>
            </v-card-actions>
        </v-card>
    </v-form>

</template>

<style scoped>
</style>