```
import helper from './helper';

import { createLocalVue, mount } from '@vue/test-utils';
import Component from '@/components/Component';

import $ from 'jquery';
global.$ = $;

import BootstrapVue from 'bootstrap-vue';
import VeeValidate from 'vee-validate';
import Password from '@/components/Password';

const localVue = createLocalVue();
localVue.use(BootstrapVue);
localVue.use(VeeValidate);

import { Validator } from 'vee-validate';

// generate custom rule
Validator.extend('mustContainUppercase', {
  getMessage: field => 'The ' + field + ' must contain an uppercase.',
  validate: value => /[A-Z]/.test(value)
});

Validator.extend('mustContainLowercase', {
  getMessage: field => 'The ' + field + ' must contain a lowercase.',
  validate: value => /[a-z]/.test(value)
});

describe('Component', () => {
  const wrapper = mount(Component, {
    localVue,
    sync: false,
    stubs: {
      Password: Password
    }
  });
  const vm = wrapper.vm;

  const password = {
    field: function() {
      return wrapper.find('[data-cy="password"]');
    },
    invalidFeedback: function() {
      return wrapper.find('[data-cy="password-invalidFeedback"]');
    }
  };

  const confirmPassword = {
    field: function() {
      return wrapper.find('[data-cy="confirmPassword"]');
    },
    invalidFeedback: function() {
      return wrapper.find('[data-cy="confirmPassword-invalidFeedback"]');
    }
  };

  it('renders correct markup', () => {
    expect(password.field().exists()).toBe(true);
    expect(confirmPassword.field().exists()).toBe(true);
  });

  it('matches snapshot', function() {
    expect(wrapper.html()).toMatchSnapshot();
  });

  describe('field validation', () => {
    it('should be required', async () => {
      await vm.validatePassword();
      helper.invalidFields([password, confirmPassword]);
      expect(password.invalidFeedback().text()).toBe('The password field is required.');
      expect(confirmPassword.invalidFeedback().text()).toBe(
        'The confirm password field is required.'
      );
    });

    describe('password field', async () => {
      it('should contain min:8 characters', async () => {
        wrapper.setData({
          form: {
            password: 'fox'
          }
        });

        await wrapper.vm.validatePassword();
        helper.invalidFields([password]);
        expect(password.invalidFeedback().text()).toBe(
          'The password field must be at least 8 characters.'
        );
      });

      it('should contain at least one uppercase letter', async () => {
        wrapper.setData({
          form: { password: 'quickbrownfox' }
        });

        await vm.validatePassword();
        helper.invalidFields([password]);

        expect(password.invalidFeedback().text()).toBe('The password must contain an uppercase.');
      });

      it('should contain at least one lowercase letter', async () => {
        wrapper.setData({
          form: { password: 'QUICKBROWNFOX' }
        });

        await vm.validatePassword();
        helper.invalidFields([password]);

        expect(password.invalidFeedback().text()).toBe('The password must contain a lowercase.');
      });
    });

    describe('confirm password field', () => {
      it('should match the password field', async () => {
        wrapper.setData({
          form: {
            password: 'Sourcefit2018',
            confirmPassword: 'Sourcefit20181'
          }
        });

        await vm.validatePassword();

        helper.invalidFields([confirmPassword]);
        expect(confirmPassword.invalidFeedback().text()).toBe('The password did not match.');
      });
    });
  });

  describe('show password toggle', function() {
    it('should be input type password', function() {
      expect(password.field().attributes('type')).toBe('password');
      expect(confirmPassword.field().attributes('type')).toBe('password');
    });

    it('should be input type text upon show password toggle', async () => {
      wrapper.find('[data-cy=toggleShowPasswordBtn]').trigger('click.prevent');
      /**
       * Computed property is not reactive in tests when returning a nestedValue of an object prop
       * that's why we need to setTimeout to wait for Vue to perform an update
       */
      await new Promise(resolve => setTimeout(resolve));
      expect(password.field().attributes('type')).toBe('text');
      expect(confirmPassword.field().attributes('type')).toBe('text');
    });
  });

```

```
import { createLocalVue, mount } from '@vue/test-utils';
import Password from '@/components/Password';
import BootstrapVue from 'bootstrap-vue';
import $ from 'jquery';

const localVue = createLocalVue();
localVue.use(BootstrapVue);
global.$ = $;

describe('Password Component', () => {
  const wrapper = mount(Password, {
    localVue
  });
  const vm = wrapper.vm;

  const passwordField = wrapper.find('[data-cy=password]');
  const passwordInvalidField = wrapper.find('[data-cy=password-invalidFeedback]');

  it('renders correct markup', () => {
    expect(passwordField.exists()).toBe(true);
    expect(passwordInvalidField.exists()).toBe(true);
  });

  it('matches snapshot', () => {
    expect(wrapper.html()).toMatchSnapshot();
  });

  it('shows error message', () => {
    wrapper.setProps({
      error: 'Invalid Password'
    });
    expect(passwordField.classes()).toContain('is-invalid');
    expect(passwordInvalidField.isVisible()).toBe(true);
    expect(passwordInvalidField.text()).toBe('Invalid Password');
  });

  it('check emitted password input value', () => {
    vm.emitValue('emit password value');
    expect(wrapper.emitted().input[0]).toContain('emit password value');
  });

  it('check emitted password passed score', () => {
    vm.emitPassed(true);
    expect(wrapper.emitted().passed[0]).toContain(true);
  });
});

```

```
let knex, Message;

beforeAll(() => {
  knex = require("../config/database"); // we need this to destory the knex initialization when importing models
  Message = require("../models").Message;
});

afterAll(() => {
  knex.destroy(); // need to destroy the knex because a method is calling knex
});

describe("Message Spec", () => {
  it('calls "all" method', async () => {
    await Message.all().then(messages => {
      expect(Array.isArray(messages)).toBe(true);

      messages.forEach(message => {
        expect(Object.keys(message)).toEqual(["user_id", "message"]);
        expect(typeof message.user_id).toEqual("string");
        expect(typeof message.message).toEqual("string");
      });
    });
  });
  it('calls "lazyLoad" methood', async () => {
    await Message.lazyLoad().then(([total, messages, paginate]) => {
      expect(typeof total.count).toEqual("number");
      expect(Array.isArray(messages)).toBe(true);
      expect(typeof paginate).toEqual("object");

      messages.forEach(message => {
        expect(Object.keys(message)).toEqual([
          "id",
          "user_id",
          "message",
          "created_at",
          "updated_at"
        ]);
        expect(typeof message.id).toEqual("number");
        expect(typeof message.user_id).toEqual("string");
        expect(typeof message.message).toEqual("string");
        expect(Date.parse(message.created_at)).not.toBe("NaN");
        expect(Date.parse(message.updated_at)).not.toBe("NaN");
      });
    });
  });
});

```