<template>
  <div class="card">
    <div class="card-header">{{title}}</div>
    <div class="card-body">
      <div class="alert alert-danger" v-show="postError">{{ postErrorMessage }}</div>
      <form @submit.prevent="onSubmit">
        <div class="form-group row">
          <label for="userName" class="col-sm-3 col-form-label">User Name</label>
          <div class="col-sm-9">
            <input
              id="userName"
              name="userName"
              class="form-control"
              v-model="user.userName"
              :class="{'is-invalid': isSubmitted && $v.user.userName.$invalid}"
              @input="$v.user.userName.$touch()"
            />
            <span class="invalid-feedback">
              <span v-if="!$v.user.userName.required">Enter an user name</span>
              <span
                v-if="!$v.user.userName.minLength"
              >The user name must be longer than 3 characters.</span>
            </span>
          </div>
        </div>
        <div class="form-group row">
          <label for="email" class="col-sm-3 col-form-label">Email</label>
          <div class="col-sm-9">
            <input
              id="email"
              name="email"
              class="form-control"
              v-model="user.email"
              :class="{'is-invalid': isSubmitted && $v.user.email.$invalid}"
              @input="$v.user.email.$touch()"
            />
            <span class="invalid-feedback">
              <span v-if="!$v.user.email.required">Enter an email</span>
              <span v-if="!$v.user.email.minLength">The email must be longer than 3 characters.</span>
            </span>
          </div>
        </div>
        <div class="form-group row">
          <label for="emailConfirmed" class="col-sm-3 col-form-label">Email Confirmed</label>
          <div class="col-sm-9">
            <input
              type="checkbox"
              id="emailConfirmed"
              name="emailConfirmed"
              v-model="user.emailConfirmed"
            />
          </div>
        </div>
        <div class="form-group row">
          <label for="phoneNumber" class="col-sm-3 col-form-label">Phone Number</label>
          <div class="col-sm-9">
            <input
              id="phoneNumber"
              name="phoneNumber"
              class="form-control"
              v-model="user.phoneNumber"
            />
            <span class="invalid-feedback">Enter a phone number</span>
          </div>
        </div>
        <div class="form-group row">
          <label for="phoneNumberConfirmed" class="col-sm-3 col-form-label">Phone Number Confirmed</label>
          <div class="col-sm-9">
            <input
              type="checkbox"
              id="phoneNumberConfirmed"
              name="phoneNumberConfirmed"
              v-model="user.phoneNumberConfirmed"
            />
          </div>
        </div>
        <div class="form-group row">
          <label for="twoFactorEnabled" class="col-sm-3 col-form-label">Two Factor Enabled</label>
          <div class="col-sm-9">
            <input
              type="checkbox"
              id="twoFactorEnabled"
              name="twoFactorEnabled"
              v-model="user.twoFactorEnabled"
            />
          </div>
        </div>
        <div class="form-group row">
          <label for="lockoutEnabled" class="col-sm-3 col-form-label">Lockout Enabled</label>
          <div class="col-sm-9">
            <input
              type="checkbox"
              id="lockoutEnabled"
              name="lockoutEnabled"
              v-model="user.lockoutEnabled"
            />
          </div>
        </div>
        <div class="form-group row">
          <label for="accessFailedCount" class="col-sm-3 col-form-label">Access Failed Count</label>
          <div class="col-sm-9">
            <input
              type="number"
              id="accessFailedCount"
              name="accessFailedCount"
              class="form-control"
              v-model="user.accessFailedCount"
            />
          </div>
        </div>
        <div class="form-group row">
          <label for="lockoutEnd" class="col-sm-3 col-form-label">Lockout End</label>
          <div class="col-sm-9">
            <b-form-datepicker
              id="example-datepicker"
              v-model="user.lockoutEndDate"
              class="mb-2"
              today-button
              reset-button
              close-button
            ></b-form-datepicker>
            <b-form-timepicker v-model="user.lockoutEndTime" now-button reset-button></b-form-timepicker>
          </div>
        </div>
        <div class="form-group row">
          <label for="description" class="col-sm-3 col-form-label"></label>
          <div class="col-sm-9">
            <button class="btn btn-primary">Save</button>
          </div>
        </div>
      </form>
    </div>
    <div class="card-footer">
      <router-link class="btn btn-outline-secondary" to="/users" style="width:80px">
        <i class="fa fa-chevron-left"></i> Back
      </router-link>
    </div>
  </div>
</template>

<script>
import { required, minLength, maxLength } from "vuelidate/lib/validators";

import axios from "./axios";

export default {
  data() {
    return {
      user: { userName: "", email: "" },
      postError: false,
      postErrorMessage: "",
      isSubmitted: false,
    };
  },
  computed: {
    title() {
      return this.$route.params.id ? "Edit User" : "Add User";
    },
    id() {
      return this.$route.params.id;
    },
  },
  validations: {
    user: {
      userName: {
        required,
        minLength: minLength(3),
      },
      email: {
        required,
        minLength: minLength(3),
      },
    },
  },
  methods: {
    onSubmit() {
      this.isSubmitted = true;
      if (this.$v.user.$invalid) {
        return;
      }

      this.user.lockoutEnd = this.user.lockoutEndDate;
      if (this.user.lockoutEndDate && this.user.lockoutEndTime) {
        this.user.lockoutEnd += "T" + this.user.lockoutEndTime;
      }

      this.user.lockoutEnd = this.user.lockoutEnd ? this.user.lockoutEnd : null;
      this.user.accessFailedCount = this.user.accessFailedCount
        ? parseInt(this.user.accessFailedCount)
        : 0;

      const promise = this.id
        ? axios.put(this.id, this.user)
        : axios.post("", this.user);

      promise.then((rs) => {
        const id = this.id ? this.id : rs.data.id;
        this.$router.push("/users/" + id);
      });
    },
  },
  created() {
    const id = this.$route.params.id;
    if (id) {
      axios.get(id).then((rs) => {
        this.user = rs.data;

        if (this.user.lockoutEnd) {
          var lockoutEnd = new Date(this.user.lockoutEnd);

          this.user.lockoutEndDate = [
            lockoutEnd.getFullYear(),
            "-",
            ("0" + (lockoutEnd.getMonth() + 1)).slice(-2),
            "-",
            ("0" + lockoutEnd.getDate()).slice(-2),
          ].join("");
          this.user.lockoutEndTime = [
            ("0" + lockoutEnd.getHours()).slice(-2),
            ":",
            ("0" + lockoutEnd.getMinutes()).slice(-2),
            ":",
            ("0" + lockoutEnd.getSeconds()).slice(-2),
          ].join("");
          console.log(this.user.lockoutEndTime);
        }
      });
    }
  },
};
</script>

<style scoped>
.field-error {
  border: 1px solid red;
}

.col-form-label {
  text-align: right;
}
</style>