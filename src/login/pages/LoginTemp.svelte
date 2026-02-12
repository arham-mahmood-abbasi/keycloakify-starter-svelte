<script lang="ts">
  import PasswordWrapper from '@keycloakify/svelte/login/components/PasswordWrapper.svelte';
  import type { PageProps } from '@keycloakify/svelte/login/pages/PageProps';
  import { useState } from '@keycloakify/svelte/tools/useState';
  import { kcSanitize } from 'keycloakify/lib/kcSanitize';
  import { getKcClsx } from 'keycloakify/login/lib/kcClsx';
  import { clsx } from 'keycloakify/tools/clsx';
  import type { I18n } from '../i18n/i18n';
  import type { KcContext } from '../KcContext';
  const {
    kcContext,
    i18n,
    doUseDefaultCss,
    Template,
    classes,
  }: PageProps<Extract<KcContext, { pageId: 'login.ftl' }>, I18n> = $props();
  const { kcClsx } = getKcClsx({
    doUseDefaultCss,
    classes,
  });

  const { social, realm, url, usernameHidden, login, auth, registrationDisabled, messagesPerField } = kcContext;

  const { msg, msgStr } = $i18n;

  const [isLoginButtonDisabled, setIsLoginButtonDisabled] = useState(false);
 
</script>

{#snippet socialProvidersSnippet()}
    {@const providers = social?.providers?.filter(p => p.alias === 'google')}
    {#if realm.password && providers && providers.length > 0}
        <div id="kc-social-providers" >
            <div class="relative flex items-center py-4">
                <div class="grow border-t border-gray-300"></div>
                <span class="mx-4 text-[#360940] text-sm uppercase tracking-widest">
                    Or
                </span>
                <div class="grow border-t border-gray-300"></div>
            </div>

            <ul class="space-y-3">
                {#each providers as p (p.providerId)}
                    <li>
                        <a
                            href={p.loginUrl}
                            class="flex w-full items-center justify-center gap-2 rounded-lg border border-gray-300 py-3 hover:border-[#360940] "
                        >
                            <img src="https://www.gstatic.com/images/branding/product/1x/gsa_512dp.png" class="h-7 w-7" />
                            <span class="text-md font-medium text-black">
                                Continue with Google
                            </span>
                        </a>
                    </li>
                {/each}
            </ul>
        </div>
    {/if}
{/snippet}


<Template
  {kcContext}
  {i18n}
  {doUseDefaultCss}
  {classes}
  displayMessage={!messagesPerField.existsError('username', 'password')}
  displayInfo={false}
  socialProvidersNode={socialProvidersSnippet}
>
  {#snippet headerNode()}
    {@render msg('loginAccountTitle')()}
  {/snippet}

  <!-- {#snippet infoNode()}
    <div id="kc-registration-container">
      <div id="kc-registration">
        <span>
          {@render msg('noAccount')()}&nbsp;
          <a
            tabindex={8}
            href={url.registrationUrl}
          >
            {@render msg('doRegister')()}
          </a>
        </span>
      </div>
    </div>
  {/snippet} -->
  <div id="kc-form">
    <div id="kc-form-wrapper">
      {#if realm.password}
        <form
          id="kc-form-login"
          onsubmit={() => {
            setIsLoginButtonDisabled(true);
          }}
          action={url.loginAction}
          method="post"
        >
          {#if !usernameHidden}
            <div class={kcClsx('kcFormGroupClass')}>
              <label
                for="username"
                class={kcClsx('kcLabelClass')}
              >
                {@render (!realm.loginWithEmailAllowed
                  ? msg('username')
                  : !realm.registrationEmailAsUsername
                    ? msg('usernameOrEmail')
                    : msg('email'))()}
              </label>
              <!-- svelte-ignore a11y_autofocus -->
              <input
                tabindex={2}
                id="username"
                class={clsx(
    kcClsx('kcInputClass'),
    /* Only show red if username has an error */
    messagesPerField.existsError('username') && "border-red-500 ring-1 ring-red-500"
  )}
                placeholder={msgStr('username.placeholder')}
                name="username"
                value={login.username ?? ''}
                type="text"
                autofocus
                autocomplete="username"
                aria-invalid={messagesPerField.existsError('username')}
              />
              {#if messagesPerField.existsError('username')}
                <div
                  id="input-error-username"
                  class={kcClsx('kcInputErrorMessageClass')}
                  aria-live="polite">{@html kcSanitize(messagesPerField.getFirstError('username'))}</div
                >
              {/if}
            </div>
          {/if}

          <div class={clsx(kcClsx('kcFormGroupClass'), 'my-0 py-0 flex-col')}>
            <div></div>
            <label
              for="password"
              class={kcClsx('kcLabelClass')}
            >
              {@render msg('password')()}
            </label>
            <PasswordWrapper
              {kcClsx}
              {i18n}
              passwordInputId="password"
            >
              <input
                tabindex={3}
                id="password"
                class={clsx(
                kcClsx('kcInputClass'),
                messagesPerField.existsError('password') && "border-red-500 ring-1 ring-red-500"
                )}
                placeholder={msgStr('password.placeholder.login')}
                name="password"
                type="password"
                autocomplete="current-password"
                aria-invalid={messagesPerField.existsError('password')}
              />
            </PasswordWrapper>
              {#if messagesPerField.existsError('password')}
              <div
                id="input-error-password"
                class={kcClsx('kcInputErrorMessageClass')}
                aria-live="polite">{@html kcSanitize(messagesPerField.getFirstError('password'))}</div
              >
            {/if}
          </div>

          <!-- <div class={kcClsx('kcFormGroupClass', 'kcFormSettingClass')}>
            <div id="kc-form-options">
              {#if realm.rememberMe && !usernameHidden}
                <div class="checkbox">
                  <label>
                    <input
                      tabindex={5}
                      id="rememberMe"
                      name="rememberMe"
                      type="checkbox"
                      checked={!!login.rememberMe}
                    />&nbsp;
                    {@render msg('rememberMe')()}
                  </label>
                </div>
              {/if}
            </div>
          </div> -->

          <div id="kc-form-buttons">
            <input
              type="hidden"
              id="id-hidden-input"
              name="credentialId"
              value={auth.selectedCredential}/>
             <div class="flex items-center justify-between w-full">
              <input
              tabindex={7}
              disabled={$isLoginButtonDisabled}
              class="bg-[#360940] my-4 py-3 w-1/2 text-center text-white text-base rounded-lg cursor-pointer transition-all"
              name="login"
              id="kc-login"
              type="submit"
              value={msgStr('doLogIn')}/>
            {#if realm.resetPasswordAllowed}
            <a tabindex={6} 
            href={url.loginResetCredentialsUrl} 
            class="text-[#696464AB] w-1/2 text-end text-base hover:text-[#360940] hover:underline transition-colors"
            >
            {@render msg('doForgotPassword')()}
           </a>
    {/if}
             </div>
          </div>
        </form>
      {/if}
    </div>
  </div>
</Template>
