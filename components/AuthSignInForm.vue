<script>
    import {validationMixin} from 'vuelidate';
    import required from 'vuelidate/lib/validators/required';
    import minLength from 'vuelidate/lib/validators/minLength';
    import maxLength from 'vuelidate/lib/validators/maxLength';
    import {login} from "~/api/index";
    import {getServerValidator, fillServerErrors, getErrorText} from "~/assets/server-error";
    import checkEmpty from '~/assets/v-check-empty';
    import {makeAccepter} from "~/assets/utils";
    import InputMaskedName from '~/components/InputMaskedName';

    export default {
        components: {
            InputMaskedName,
        },
        mixins: [validationMixin],
        directives: {
            checkEmpty,
        },
        data() {
            return {
                //@TODO common loading flag
                isFormSending: false,
                serverError: '',
                form: {
                    username: '',
                    password: '',
                },
                usernameMasked: '',
                sve: {
                    username: {invalid: false, isActual: false, message: ''},
                    password: {invalid: false, isActual: false, message: ''},
                },
            }
        },
        validations: {
            form: {
                username: {
                    required,
                    minLength: minLength(5),
                    maxLength: maxLength(32),
                    server: getServerValidator('username'),
                },
                password: {
                    required,
                    minLength: minLength(6),
                    maxLength: maxLength(100),
                    server: getServerValidator('password'),
                },
            }
        },
        methods: {
            onAcceptUsername: makeAccepter('username', true),
            submit() {
                if (this.isFormSending) {
                    return;
                }
                if (this.$v.$invalid) {
                    this.$v.$touch();
                    return;
                }
                this.isFormSending = true;

                login(this.form)
                    .then((authData) => {
                        this.$store.commit('SET_AUTH_PROFILE', authData);
                        this.$router.push('/');
                        this.isFormSending = false;
                    })
                    .catch((error) => {
                        let hasValidationErrors = fillServerErrors(error, this.sve);
                        if (!hasValidationErrors) {
                            this.serverError = getErrorText(error);
                        }
                        this.isFormSending = false;
                    });
            }
        }
    }
</script>

<template>
    <form class="panel__section" novalidate @submit.prevent="submit">
        <div class="u-grid u-grid--small u-grid--vertical-margin--small">
            <div class="u-cell u-cell--1-2">
                <label class="form-field" :class="{'is-error': $v.form.username.$error}">
                    <InputMaskedName class="form-field__input" v-check-empty
                                     @accept="onAcceptUsername"
                                     @blur.native="$v.form.username.$touch()"
                                     @input.native="sve.username.isActual = false"
                    />
                    <span class="form-field__label">Username</span>
                </label>
                <span class="form-field__error" v-if="$v.form.username.$dirty && !$v.form.username.required">Enter username</span>
                <span class="form-field__error" v-if="$v.form.username.$dirty && !$v.form.username.minLength">Username is too short</span>
                <span class="form-field__error" v-if="$v.form.username.$dirty && !$v.form.username.maxLength">Username is too long</span>
                <span class="form-field__error" v-if="$v.form.username.$dirty && !$v.form.username.server">{{ sve.username.message }}</span>
            </div>
            <div class="u-cell u-cell--1-2">
                <label class="form-field" :class="{'is-error': $v.form.password.$error}">
                    <input class="form-field__input" type="password" v-check-empty
                           v-model="form.password"
                           @blur="$v.form.password.$touch()"
                           @input="sve.password.isActual = false"
                    >
                    <span class="form-field__label">Password</span>
                </label>
                <span class="form-field__error" v-if="$v.form.password.$dirty && !$v.form.password.required">Enter password</span>
                <span class="form-field__error" v-if="$v.form.password.$dirty && !$v.form.password.minLength">Password is too short</span>
                <span class="form-field__error" v-if="$v.form.password.$dirty && !$v.form.password.maxLength">Password is too long</span>
                <span class="form-field__error" v-if="$v.form.username.$dirty && !$v.form.password.server">{{ sve.password.message }}</span>
            </div>
            <div class="u-cell">
                <button class="button button--main button--full" :class="{'is-disabled': $v.$invalid}">Sign In</button>
                <div class="form-field__error" v-if="serverError">{{ serverError }}</div>
            </div>
        </div>
    </form>
</template>