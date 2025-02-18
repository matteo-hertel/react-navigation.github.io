---
id: static-combine-with-dynamic
title: Combining static and dynamic APIs
sidebar_label: Combining with dynamic API
---

While the static API has many advantages, it doesn't fit use cases where the navigation configuration needs to be dynamic. So React Navigation supports interop between the static and dynamic APIs.

Keep in mind that the features provided by the static API such as automatic linking configuration and automatic TypeScript types need the whole configuration to be static. If part of the configuration is dynamic, you'll need to handle those parts manually.

There are 2 ways you may want to combine the static and dynamic APIs:

## Static root navigator, dynamic nested navigator

This is useful if you want to keep your configuration static, but need to use a dynamic configuration for a specific navigator.

Let's consider the following example:

- You have a root stack navigator that contains a tab navigator in a screen.
- The tab navigator is defined using the dynamic API.

Our static configuration would look like this:

```js
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const RootStack = createNativeStackNavigator({
  screens: {
    Home: {
      screen: HomeScreen,
    },
    Feed: {
      screen: FeedScreen,
      linking: {
        path: 'feed',
      },
    },
  },
});
```

Here, `FeedScreen` is a component that renders a tab navigator and is defined using the dynamic API:

```js
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

const Tab = createBottomTabNavigator();

function FeedScreen() {
  return (
    <Tab.Navigator>
      <Tab.Screen name="Latest" component={LatestScreen} />
      <Tab.Screen name="Popular" component={PopularScreen} />
    </Tab.Navigator>
  );
}
```

This code will work, but we're missing 2 things:

- Linking configuration for the screens in the top tab navigator.
- TypeScript types for the screens in the top tab navigator.

Since the nested navigator is defined using the dynamic API, we need to handle these manually. For the linking configuration, we can define the screens in the `linking` property of the `Feed` screen:

```js
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const RootStack = createNativeStackNavigator({
  screens: {
    Home: {
      screen: HomeScreen,
    },
    Feed: {
      screen: FeedScreen,
      linking: {
        path: 'feed',
        screens: {
          Latest: 'latest',
          Popular: 'popular',
        },
      },
    },
  },
});
```

Here the `screens` property is the same as how you'd define it with `linking` config with the dynamic API. It can contain configuration for any nested navigators as well. See [configuring links](configuring-links.md) for more details on the API.

For the TypeScript types, we can define the type of the `FeedScreen` component:

```tsx
import {
  StaticScreenProps,
  NavigatorScreenParams,
} from '@react-navigation/native';

type FeedParamList = {
  Latest: undefined;
  Popular: undefined;
};

type Props = StaticScreenProps<NavigatorScreenParams<FeedParamList>>;

function FeedScreen(_: Props) {
  // ...
}
```

In the above snippet:

1. We first define the param list type for screens in the navigator that defines params for each screen
2. Then we use the `NavigatorScreenParams` type to get the type of route's `params` which will include types for the nested screens
3. Finally, we use the type of `params` with `StaticScreenProps` to define the type of the screen component

This is based on how we'd define the type for a screen with a nested navigator with the dynamic API. See [Type checking screens and params in nested navigator](typescript.md#type-checking-screens-and-params-in-nested-navigator).

## Dynamic root navigator, static nested navigator

This is useful if you already have a dynamic configuration, but want to migrate to the static API. This way you can migrate one navigator at a time.

TODO
