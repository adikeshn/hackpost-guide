<script setup lang="ts">
    import type { RouteLocationRaw } from 'vue-router';
    import { hasProtocol } from 'ufo';
    
    type NuxtLinkProps = {
        to?: RouteLocationRaw;
        href?: RouteLocationRaw;
        external?: boolean;
        replace?: boolean;
        target?: '_blank' | '_self' | '_parent' | '_top';
        rel?: string;
        prefetch?: boolean;
    }

    const props = defineProps<NuxtLinkProps>();
    const NuxtLink = defineNuxtLink({
        trailingSlash: 'append',
    });

    const path = computed(() => props.to || props.href || '');

    //From 
    const isExternal = computed(() => {
        
        //external prop is explicitly set:
        if(props.external){
            return true;
        }

        //when 'target' prop is set, link is external
        if(props.target && props.target !== '_self'){
            return true;
        }

        //when 'to' prop is a route object then it's an internal link
        if(typeof path.value === 'object'){
            return false;
        }

        return hasProtocol(path.value, { acceptRelative: true });
    });

    const propsWithoutRelAndTarget = computed(() => {
        const { rel, target, ...rest } = props;
        return rest;
    })

    // TODO: Add exceptions for certain domains if needed
    const ALLOWED_DOMAINS = [
        "imaginai.dev",
    ]
    const isDomainException = computed(() => typeof path.value === 'string' && ALLOWED_DOMAINS.some(domain => (path.value as string).startsWith(domain)))
    const defaultRel = computed(() => isExternal.value && !isDomainException.value ? 'nofollow noindex noreferrer' : null)
    const relValue = computed(() => props.rel ?? defaultRel.value)

    const defaultTarget = computed(() => isExternal.value ? '_blank' : '')
    const targetValue = computed(() => props.target ?? defaultTarget.value)
    </script>

<template>
  <component :is="NuxtLink" v-bind="propsWithoutRelAndTarget" :rel="relValue" :target="targetValue">
    <slot />
  </component>
</template>