+++
title = "React Native with Expo: Complete Setup Guide for Modern Mobile Development"
date = "2025-07-28T16:00:00+05:30"
author = ""
draft = true
authorTwitter = "" #do not include @
cover = ""
tags = ["react-native", "expo", "mobile-development", "ios", "android", "javascript", "typescript", "app-development"]
keywords = ["react native expo", "mobile app development", "expo setup", "react native tutorial", "ios android development", "expo cli", "mobile development setup"]
description = "Complete guide to setting up React Native with Expo for modern mobile app development. From installation to deployment, build cross-platform mobile apps efficiently."
showFullContent = false
readingTime = false
hideComments = false
+++

React Native with Expo has revolutionized mobile app development by providing a streamlined, efficient way to build cross-platform mobile applications. This comprehensive guide will take you through everything you need to know about setting up and developing with React Native and Expo, from initial setup to app store deployment.

## What is React Native and Expo?

### React Native
React Native is a framework developed by Meta (Facebook) that allows developers to build mobile applications using React and JavaScript. It enables code sharing between iOS and Android platforms while providing native performance and platform-specific UI components.

### Expo
Expo is a platform and set of tools built around React Native that makes mobile app development faster and easier. It provides:
- **Managed Workflow**: Build apps without dealing with native code
- **Development Tools**: Hot reloading, debugging, and testing tools
- **Cloud Services**: Build, deploy, and update apps over-the-air
- **Rich SDK**: Access to device APIs and native functionality

## Why Choose React Native with Expo?

### 1. **Rapid Development**
- Hot reloading for instant feedback
- No need to set up complex native development environments
- Shared codebase for iOS and Android (up to 95% code sharing)

### 2. **Rich Ecosystem**
- Extensive library of pre-built components
- Access to native device APIs through Expo SDK
- Large community and excellent documentation

### 3. **Easy Deployment**
- Over-the-air updates without app store approval
- Simplified build and deployment process
- TestFlight and Play Store integration

### 4. **Cost-Effective**
- Single development team for multiple platforms
- Faster time-to-market
- Reduced maintenance overhead

## Development Environment Setup

### Prerequisites

```bash
# Install Node.js (LTS version recommended)
# Download from https://nodejs.org/ or use a version manager

# Verify Node.js installation
node --version  # Should be 16.x or higher
npm --version

# Install Git
git --version

# For iOS development (macOS only)
xcode-select --install

# For Android development
# Download Android Studio from https://developer.android.com/studio
```

### Install Expo CLI and Tools

```bash
# Install Expo CLI globally
npm install -g @expo/cli

# Install EAS CLI for building and deployment
npm install -g eas-cli

# Verify installations
expo --version
eas --version

# Install Expo Go app on your mobile device
# iOS: https://apps.apple.com/app/expo-go/id982107779
# Android: https://play.google.com/store/apps/details?id=host.exp.exponent
```

### Development Tools Setup

```bash
# Install useful development tools
npm install -g react-native-debugger
npm install -g flipper

# VS Code extensions (recommended)
# - React Native Tools
# - ES7+ React/Redux/React-Native snippets
# - Prettier
# - ESLint
# - Auto Rename Tag
# - Bracket Pair Colorizer
```

## Creating Your First Expo App

### Initialize New Project

```bash
# Create new Expo project
npx create-expo-app MyAwesomeApp

# Navigate to project directory
cd MyAwesomeApp

# Start development server
npx expo start

# Alternative: Create with specific template
npx create-expo-app MyApp --template blank-typescript
```

### Project Structure Overview

```
MyAwesomeApp/
├── .expo/                 # Expo configuration cache
├── assets/               # Images, fonts, and other assets
│   ├── images/
│   └── fonts/
├── components/           # Reusable UI components
├── screens/             # App screens/pages
├── navigation/          # Navigation configuration
├── services/           # API calls and external services
├── utils/              # Utility functions
├── hooks/              # Custom React hooks
├── constants/          # App constants and configuration
├── types/              # TypeScript type definitions
├── App.tsx             # Main app component
├── app.json            # Expo configuration
├── package.json        # Dependencies and scripts
├── tsconfig.json       # TypeScript configuration
└── babel.config.js     # Babel configuration
```

### Basic App Configuration

```json
// app.json
{
  "expo": {
    "name": "My Awesome App",
    "slug": "my-awesome-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "light",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "assetBundlePatterns": [
      "**/*"
    ],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.yourcompany.myawesomeapp"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#FFFFFF"
      },
      "package": "com.yourcompany.myawesomeapp"
    },
    "web": {
      "favicon": "./assets/favicon.png"
    },
    "plugins": [
      "expo-font",
      "expo-camera",
      "expo-location"
    ]
  }
}
```

## Building Your First App

### Main App Component

```typescript
// App.tsx
import React from 'react';
import { StatusBar } from 'expo-status-bar';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { SafeAreaProvider } from 'react-native-safe-area-context';

import HomeScreen from './screens/HomeScreen';
import ProfileScreen from './screens/ProfileScreen';
import { RootStackParamList } from './types/navigation';

const Stack = createNativeStackNavigator<RootStackParamList>();

export default function App() {
  return (
    <SafeAreaProvider>
      <NavigationContainer>
        <Stack.Navigator initialRouteName="Home">
          <Stack.Screen
            name="Home"
            component={HomeScreen}
            options={{ title: 'My Awesome App' }}
          />
          <Stack.Screen
            name="Profile"
            component={ProfileScreen}
            options={{ title: 'Profile' }}
          />
        </Stack.Navigator>
        <StatusBar style="auto" />
      </NavigationContainer>
    </SafeAreaProvider>
  );
}
```

### Home Screen Component

```typescript
// screens/HomeScreen.tsx
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  FlatList,
  Alert,
  RefreshControl,
} from 'react-native';
import { NativeStackNavigationProp } from '@react-navigation/native-stack';
import { SafeAreaView } from 'react-native-safe-area-context';

import { RootStackParamList } from '../types/navigation';
import { TaskItem } from '../components/TaskItem';
import { AddTaskModal } from '../components/AddTaskModal';
import { Task } from '../types/task';

type HomeScreenNavigationProp = NativeStackNavigationProp<
  RootStackParamList,
  'Home'
>;

interface Props {
  navigation: HomeScreenNavigationProp;
}

export default function HomeScreen({ navigation }: Props) {
  const [tasks, setTasks] = useState<Task[]>([]);
  const [isModalVisible, setIsModalVisible] = useState(false);
  const [refreshing, setRefreshing] = useState(false);

  useEffect(() => {
    loadTasks();
  }, []);

  const loadTasks = async () => {
    // Simulate API call
    const mockTasks: Task[] = [
      { id: '1', title: 'Learn React Native', completed: false },
      { id: '2', title: 'Build awesome app', completed: false },
      { id: '3', title: 'Deploy to app stores', completed: false },
    ];
    setTasks(mockTasks);
  };

  const addTask = (title: string) => {
    const newTask: Task = {
      id: Date.now().toString(),
      title,
      completed: false,
    };
    setTasks([...tasks, newTask]);
    setIsModalVisible(false);
  };

  const toggleTask = (id: string) => {
    setTasks(tasks.map(task =>
      task.id === id ? { ...task, completed: !task.completed } : task
    ));
  };

  const deleteTask = (id: string) => {
    Alert.alert(
      'Delete Task',
      'Are you sure you want to delete this task?',
      [
        { text: 'Cancel', style: 'cancel' },
        {
          text: 'Delete',
          style: 'destructive',
          onPress: () => setTasks(tasks.filter(task => task.id !== id))
        },
      ]
    );
  };

  const onRefresh = async () => {
    setRefreshing(true);
    await loadTasks();
    setRefreshing(false);
  };

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>My Tasks</Text>
        <TouchableOpacity
          style={styles.profileButton}
          onPress={() => navigation.navigate('Profile')}
        >
          <Text style={styles.profileButtonText}>Profile</Text>
        </TouchableOpacity>
      </View>

      <FlatList
        data={tasks}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <TaskItem
            task={item}
            onToggle={toggleTask}
            onDelete={deleteTask}
          />
        )}
        refreshControl={
          <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
        }
        style={styles.list}
        showsVerticalScrollIndicator={false}
      />

      <TouchableOpacity
        style={styles.addButton}
        onPress={() => setIsModalVisible(true)}
      >
        <Text style={styles.addButtonText}>+</Text>
      </TouchableOpacity>

      <AddTaskModal
        visible={isModalVisible}
        onClose={() => setIsModalVisible(false)}
        onAdd={addTask}
      />
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingHorizontal: 20,
    paddingVertical: 15,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
  },
  profileButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 15,
    paddingVertical: 8,
    borderRadius: 20,
  },
  profileButtonText: {
    color: '#fff',
    fontWeight: '600',
  },
  list: {
    flex: 1,
    paddingHorizontal: 20,
  },
  addButton: {
    position: 'absolute',
    bottom: 30,
    right: 30,
    width: 60,
    height: 60,
    borderRadius: 30,
    backgroundColor: '#007AFF',
    justifyContent: 'center',
    alignItems: 'center',
    elevation: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 4,
  },
  addButtonText: {
    fontSize: 30,
    color: '#fff',
    fontWeight: '300',
  },
});
```

### Reusable Components

```typescript
// components/TaskItem.tsx
import React from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  Animated,
} from 'react-native';
import { Swipeable } from 'react-native-gesture-handler';
import { Ionicons } from '@expo/vector-icons';

import { Task } from '../types/task';

interface Props {
  task: Task;
  onToggle: (id: string) => void;
  onDelete: (id: string) => void;
}

export function TaskItem({ task, onToggle, onDelete }: Props) {
  const renderRightActions = (
    progress: Animated.AnimatedAddition,
    dragX: Animated.AnimatedAddition
  ) => {
    const trans = dragX.interpolate({
      inputRange: [-100, -50, 0],
      outputRange: [0, 50, 100],
      extrapolate: 'clamp',
    });

    return (
      <View style={styles.rightActions}>
        <Animated.View style={[styles.deleteAction, { transform: [{ translateX: trans }] }]}>
          <TouchableOpacity
            style={styles.deleteButton}
            onPress={() => onDelete(task.id)}
          >
            <Ionicons name="trash" size={24} color="#fff" />
          </TouchableOpacity>
        </Animated.View>
      </View>
    );
  };

  return (
    <Swipeable renderRightActions={renderRightActions}>
      <View style={styles.container}>
        <TouchableOpacity
          style={styles.checkbox}
          onPress={() => onToggle(task.id)}
        >
          {task.completed && (
            <Ionicons name="checkmark" size={20} color="#007AFF" />
          )}
        </TouchableOpacity>

        <Text style={[
          styles.title,
          task.completed && styles.completedTitle
        ]}>
          {task.title}
        </Text>
      </View>
    </Swipeable>
  );
}

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: '#fff',
    paddingHorizontal: 20,
    paddingVertical: 15,
    marginVertical: 5,
    borderRadius: 10,
    elevation: 2,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.1,
    shadowRadius: 2,
  },
  checkbox: {
    width: 24,
    height: 24,
    borderRadius: 12,
    borderWidth: 2,
    borderColor: '#007AFF',
    marginRight: 15,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    flex: 1,
    fontSize: 16,
    color: '#333',
  },
  completedTitle: {
    textDecorationLine: 'line-through',
    color: '#999',
  },
  rightActions: {
    alignItems: 'center',
    justifyContent: 'center',
    width: 80,
  },
  deleteAction: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  deleteButton: {
    backgroundColor: '#FF3B30',
    width: 60,
    height: 60,
    borderRadius: 30,
    justifyContent: 'center',
    alignItems: 'center',
  },
});
```

```typescript
// components/AddTaskModal.tsx
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  Modal,
  StyleSheet,
  KeyboardAvoidingView,
  Platform,
} from 'react-native';

interface Props {
  visible: boolean;
  onClose: () => void;
  onAdd: (title: string) => void;
}

export function AddTaskModal({ visible, onClose, onAdd }: Props) {
  const [title, setTitle] = useState('');

  const handleAdd = () => {
    if (title.trim()) {
      onAdd(title.trim());
      setTitle('');
    }
  };

  const handleClose = () => {
    setTitle('');
    onClose();
  };

  return (
    <Modal
      visible={visible}
      animationType="slide"
      presentationStyle="pageSheet"
      onRequestClose={handleClose}
    >
      <KeyboardAvoidingView
        style={styles.container}
        behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      >
        <View style={styles.header}>
          <TouchableOpacity onPress={handleClose}>
            <Text style={styles.cancelButton}>Cancel</Text>
          </TouchableOpacity>
          <Text style={styles.title}>Add Task</Text>
          <TouchableOpacity onPress={handleAdd}>
            <Text style={[styles.addButton, !title.trim() && styles.disabled]}>
              Add
            </Text>
          </TouchableOpacity>
        </View>

        <View style={styles.content}>
          <TextInput
            style={styles.input}
            placeholder="Enter task title..."
            value={title}
            onChangeText={setTitle}
            autoFocus
            multiline
            maxLength={100}
          />
        </View>
      </KeyboardAvoidingView>
    </Modal>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingHorizontal: 20,
    paddingVertical: 15,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  title: {
    fontSize: 18,
    fontWeight: '600',
    color: '#333',
  },
  cancelButton: {
    fontSize: 16,
    color: '#007AFF',
  },
  addButton: {
    fontSize: 16,
    color: '#007AFF',
    fontWeight: '600',
  },
  disabled: {
    color: '#999',
  },
  content: {
    flex: 1,
    padding: 20,
  },
  input: {
    backgroundColor: '#fff',
    borderRadius: 10,
    padding: 15,
    fontSize: 16,
    minHeight: 100,
    textAlignVertical: 'top',
  },
});
```

### Type Definitions

```typescript
// types/navigation.ts
export type RootStackParamList = {
  Home: undefined;
  Profile: undefined;
};

// types/task.ts
export interface Task {
  id: string;
  title: string;
  completed: boolean;
  createdAt?: Date;
  updatedAt?: Date;
}
```

## Essential Dependencies and Setup

### Install Navigation Dependencies

```bash
# React Navigation
npm install @react-navigation/native @react-navigation/native-stack

# Required dependencies for Expo
npx expo install react-native-screens react-native-safe-area-context

# Optional: Bottom tabs, drawer navigation
npm install @react-navigation/bottom-tabs @react-navigation/drawer
npx expo install react-native-gesture-handler react-native-reanimated
```

### Install UI and Utility Libraries

```bash
# UI Components
npm install react-native-elements react-native-vector-icons
npx expo install expo-font

# State Management
npm install @reduxjs/toolkit react-redux
# or
npm install zustand

# HTTP Client
npm install axios

# Form Handling
npm install react-hook-form

# Date/Time
npm install date-fns
# or
npm install moment

# Async Storage
npx expo install @react-native-async-storage/async-storage

# Image Picker
npx expo install expo-image-picker

# Camera
npx expo install expo-camera

# Location
npx expo install expo-location

# Notifications
npx expo install expo-notifications
```

### Development Dependencies

```bash
# TypeScript
npm install -D typescript @types/react @types/react-native

# Testing
npm install -D jest @testing-library/react-native @testing-library/jest-native

# Linting and Formatting
npm install -D eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser
npm install -D prettier eslint-config-prettier eslint-plugin-prettier

# Husky for Git hooks
npm install -D husky lint-staged
```

## Advanced Features Implementation

### State Management with Redux Toolkit

```typescript
// store/store.ts
import { configureStore } from '@reduxjs/toolkit';
import tasksReducer from './tasksSlice';
import userReducer from './userSlice';

export const store = configureStore({
  reducer: {
    tasks: tasksReducer,
    user: userReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

```typescript
// store/tasksSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { Task } from '../types/task';
import * as TaskService from '../services/taskService';

interface TasksState {
  tasks: Task[];
  loading: boolean;
  error: string | null;
}

const initialState: TasksState = {
  tasks: [],
  loading: false,
  error: null,
};

export const fetchTasks = createAsyncThunk(
  'tasks/fetchTasks',
  async () => {
    const response = await TaskService.getTasks();
    return response.data;
  }
);

export const addTask = createAsyncThunk(
  'tasks/addTask',
  async (title: string) => {
    const response = await TaskService.createTask({ title });
    return response.data;
  }
);

const tasksSlice = createSlice({
  name: 'tasks',
  initialState,
  reducers: {
    toggleTask: (state, action: PayloadAction<string>) => {
      const task = state.tasks.find(t => t.id === action.payload);
      if (task) {
        task.completed = !task.completed;
      }
    },
    deleteTask: (state, action: PayloadAction<string>) => {
      state.tasks = state.tasks.filter(t => t.id !== action.payload);
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchTasks.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchTasks.fulfilled, (state, action) => {
        state.loading = false;
        state.tasks = action.payload;
      })
      .addCase(fetchTasks.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Failed to fetch tasks';
      })
      .addCase(addTask.fulfilled, (state, action) => {
        state.tasks.push(action.payload);
      });
  },
});

export const { toggleTask, deleteTask } = tasksSlice.actions;
export default tasksSlice.reducer;
```

### API Service Layer

```typescript
// services/api.ts
import axios from 'axios';
import AsyncStorage from '@react-native-async-storage/async-storage';

const API_BASE_URL = 'https://api.yourapp.com';

const api = axios.create({
  baseURL: API_BASE_URL,
  timeout: 10000,
});

// Request interceptor to add auth token
api.interceptors.request.use(
  async (config) => {
    const token = await AsyncStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor for error handling
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      // Handle token expiration
      await AsyncStorage.removeItem('authToken');
      // Navigate to login screen
    }
    return Promise.reject(error);
  }
);

export default api;
```

```typescript
// services/taskService.ts
import api from './api';
import { Task } from '../types/task';

export const getTasks = () => {
  return api.get<Task[]>('/tasks');
};

export const createTask = (task: Partial<Task>) => {
  return api.post<Task>('/tasks', task);
};

export const updateTask = (id: string, task: Partial<Task>) => {
  return api.put<Task>(`/tasks/${id}`, task);
};

export const deleteTask = (id: string) => {
  return api.delete(`/tasks/${id}`);
};
```

### Custom Hooks

```typescript
// hooks/useAsyncStorage.ts
import { useState, useEffect } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

export function useAsyncStorage<T>(key: string, defaultValue: T) {
  const [value, setValue] = useState<T>(defaultValue);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadValue();
  }, [key]);

  const loadValue = async () => {
    try {
      const stored = await AsyncStorage.getItem(key);
      if (stored !== null) {
        setValue(JSON.parse(stored));
      }
    } catch (error) {
      console.error('Error loading from AsyncStorage:', error);
    } finally {
      setLoading(false);
    }
  };

  const storeValue = async (newValue: T) => {
    try {
      setValue(newValue);
      await AsyncStorage.setItem(key, JSON.stringify(newValue));
    } catch (error) {
      console.error('Error storing to AsyncStorage:', error);
    }
  };

  const removeValue = async () => {
    try {
      setValue(defaultValue);
      await AsyncStorage.removeItem(key);
    } catch (error) {
      console.error('Error removing from AsyncStorage:', error);
    }
  };

  return { value, loading, storeValue, removeValue };
}
```

```typescript
// hooks/useCamera.ts
import { useState } from 'react';
import * as ImagePicker from 'expo-image-picker';
import { Alert } from 'react-native';

export function useCamera() {
  const [loading, setLoading] = useState(false);

  const requestPermissions = async () => {
    const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync();
    if (status !== 'granted') {
      Alert.alert('Permission needed', 'Camera roll permissions are required!');
      return false;
    }
    return true;
  };

  const pickImage = async () => {
    const hasPermission = await requestPermissions();
    if (!hasPermission) return null;

    setLoading(true);
    try {
      const result = await ImagePicker.launchImageLibraryAsync({
        mediaTypes: ImagePicker.MediaTypeOptions.Images,
        allowsEditing: true,
        aspect: [4, 3],
        quality: 1,
      });

      if (!result.canceled) {
        return result.assets[0];
      }
      return null;
    } catch (error) {
      console.error('Error picking image:', error);
      return null;
    } finally {
      setLoading(false);
    }
  };

  const takePhoto = async () => {
    const { status } = await ImagePicker.requestCameraPermissionsAsync();
    if (status !== 'granted') {
      Alert.alert('Permission needed', 'Camera permissions are required!');
      return null;
    }

    setLoading(true);
    try {
      const result = await ImagePicker.launchCameraAsync({
        allowsEditing: true,
        aspect: [4, 3],
        quality: 1,
      });

      if (!result.canceled) {
        return result.assets[0];
      }
      return null;
    } catch (error) {
      console.error('Error taking photo:', error);
      return null;
    } finally {
      setLoading(false);
    }
  };

  return { pickImage, takePhoto, loading };
}
```

## Testing Setup

### Jest Configuration

```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "jest": {
    "preset": "jest-expo",
    "setupFilesAfterEnv": ["<rootDir>/jest-setup.js"],
    "transformIgnorePatterns": [
      "node_modules/(?!((jest-)?react-native|@react-native(-community)?)|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-navigation|@react-navigation/.*|@unimodules/.*|unimodules|sentry-expo|native-base|react-native-svg)"
    ]
  }
}
```

```javascript
// jest-setup.js
import '@testing-library/jest-native/extend-expect';

// Mock AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () =>
  require('@react-native-async-storage/async-storage/jest/async-storage-mock')
);

// Mock react-native-gesture-handler
jest.mock('react-native-gesture-handler', () => {
  const View = require('react-native/Libraries/Components/View/View');
  return {
    Swipeable: View,
    DrawerLayout: View,
    State: {},
    ScrollView: View,
    Slider: View,
    Switch: View,
    TextInput: View,
    ToolbarAndroid: View,
    ViewPagerAndroid: View,
    DrawerLayoutAndroid: View,
    WebView: View,
    NativeViewGestureHandler: View,
    TapGestureHandler: View,
    FlingGestureHandler: View,
    ForceTouchGestureHandler: View,
    LongPressGestureHandler: View,
    PanGestureHandler: View,
    PinchGestureHandler: View,
    RotationGestureHandler: View,
    RawButton: View,
    BaseButton: View,
    RectButton: View,
    BorderlessButton: View,
    FlatList: View,
    gestureHandlerRootHOC: jest.fn(),
    Directions: {},
  };
});
```

### Component Testing

```typescript
// __tests__/TaskItem.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import { TaskItem } from '../components/TaskItem';
import { Task } from '../types/task';

const mockTask: Task = {
  id: '1',
  title: 'Test Task',
  completed: false,
};

describe('TaskItem', () => {
  const mockOnToggle = jest.fn();
  const mockOnDelete = jest.fn();

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('renders task title correctly', () => {
    const { getByText } = render(
      <TaskItem
        task={mockTask}
        onToggle={mockOnToggle}
        onDelete={mockOnDelete}
      />
    );

    expect(getByText('Test Task')).toBeTruthy();
  });

  it('calls onToggle when checkbox is pressed', () => {
    const { getByTestId } = render(
      <TaskItem
        task={mockTask}
        onToggle={mockOnToggle}
        onDelete={mockOnDelete}
      />
    );

    fireEvent.press(getByTestId('checkbox'));
    expect(mockOnToggle).toHaveBeenCalledWith('1');
  });

  it('shows completed style when task is completed', () => {
    const completedTask = { ...mockTask, completed: true };
    const { getByText } = render(
      <TaskItem
        task={completedTask}
        onToggle={mockOnToggle}
        onDelete={mockOnDelete}
      />
    );

    const titleElement = getByText('Test Task');
    expect(titleElement.props.style).toContainEqual(
      expect.objectContaining({
        textDecorationLine: 'line-through',
      })
    );
  });
});
```

## Building and Deployment

### EAS Build Setup

```bash
# Login to Expo account
eas login

# Configure EAS Build
eas build:configure

# This creates eas.json configuration file
```

```json
// eas.json
{
  "cli": {
    "version": ">= 3.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": {
        "resourceClass": "m1-medium"
      }
    },
    "preview": {
      "distribution": "internal",
      "ios": {
        "simulator": true
      }
    },
    "production": {
      "ios": {
        "resourceClass": "m1-medium"
      }
    }
  },
  "submit": {
    "production": {}
  }
}
```

### Build Commands

```bash
# Build for development
eas build --profile development --platform ios
eas build --profile development --platform android

# Build for production
eas build --profile production --platform all

# Build for specific platform
eas build --profile production --platform ios
eas build --profile production --platform android

# Check build status
eas build:list
```

### App Store Submission

```bash
# Configure app store submission
eas submit:configure

# Submit to App Store (iOS)
eas submit --platform ios

# Submit to Google Play (Android)
eas submit --platform android

# Check submission status
eas submit:list
```

### Over-the-Air Updates

```bash
# Install EAS Update
npx expo install expo-updates

# Configure updates in app.json
```

```json
// app.json - Add updates configuration
{
  "expo": {
    "updates": {
      "url": "https://u.expo.dev/[your-project-id]"
    },
    "runtimeVersion": {
      "policy": "sdkVersion"
    }
  }
}
```

```bash
# Publish update
eas update --branch production --message "Bug fixes and improvements"

# Publish to specific branch
eas update --branch staging --message "New features for testing"
```

## Performance Optimization

### Bundle Size Optimization

```javascript
// metro.config.js
const { getDefaultConfig } = require('expo/metro-config');

const config = getDefaultConfig(__dirname);

// Enable tree shaking
config.resolver.platforms = ['native', 'ios', 'android'];

// Optimize bundle size
config.transformer.minifierConfig = {
  keep_fnames: true,
  mangle: {
    keep_fnames: true,
  },
};

module.exports = config;
```

### Image Optimization

```typescript
// utils/imageUtils.ts
import { manipulateAsync, SaveFormat } from 'expo-image-manipulator';

export const optimizeImage = async (uri: string, maxWidth = 800) => {
  try {
    const result = await manipulateAsync(
      uri,
      [{ resize: { width: maxWidth } }],
      { compress: 0.8, format: SaveFormat.JPEG }
    );
    return result.uri;
  } catch (error) {
    console.error('Error optimizing image:', error);
    return uri;
  }
};
```

### Memory Management

```typescript
// hooks/useMemoryOptimization.ts
import { useEffect } from 'react';
import { AppState } from 'react-native';

export function useMemoryOptimization() {
  useEffect(() => {
    const handleAppStateChange = (nextAppState: string) => {
      if (nextAppState === 'background') {
        // Clear caches, cancel network requests, etc.
        console.log('App went to background, cleaning up...');
      }
    };

    const subscription = AppState.addEventListener('change', handleAppStateChange);

    return () => {
      subscription?.remove();
    };
  }, []);
}
```

## Debugging and Development Tools

### Flipper Integration

```bash
# Install Flipper desktop app
# Download from https://fbflipper.com/

# Install React Native Flipper plugin
npm install --save-dev react-native-flipper
```

### React Native Debugger

```bash
# Install React Native Debugger
# Download from https://github.com/jhen0409/react-native-debugger

# Enable remote debugging in Expo Dev Tools
# Press 'd' in terminal or shake device and select "Debug Remote JS"
```

### Expo Dev Tools

```bash
# Start with dev tools
npx expo start --dev-client

# Open in browser
npx expo start --web

# Clear cache
npx expo start --clear

# Production mode
npx expo start --no-dev --minify
```

## Best Practices and Tips

### Project Structure Best Practices

```
src/
├── components/          # Reusable UI components
│   ├── common/         # Generic components
│   └── forms/          # Form-specific components
├── screens/            # Screen components
├── navigation/         # Navigation configuration
├── services/          # API and external services
├── store/             # State management
├── hooks/             # Custom React hooks
├── utils/             # Utility functions
├── constants/         # App constants
├── types/             # TypeScript definitions
└── assets/            # Images, fonts, etc.
```

### Code Quality Setup

```json
// .eslintrc.js
module.exports = {
  extends: [
    'expo',
    '@react-native-community',
    'prettier',
  ],
  plugins: ['prettier'],
  rules: {
    'prettier/prettier': 'error',
    'react-native/no-unused-styles': 'error',
    'react-native/split-platform-components': 'error',
    'react-native/no-inline-styles': 'warn',
  },
};
```

```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2
}
```

### Performance Monitoring

```typescript
// utils/performance.ts
import { InteractionManager } from 'react-native';

export const measurePerformance = (name: string, fn: () => void) => {
  const start = Date.now();

  InteractionManager.runAfterInteractions(() => {
    fn();
    const end = Date.now();
    console.log(`${name} took ${end - start}ms`);
  });
};

export const deferExpensiveWork = (work: () => void) => {
  InteractionManager.runAfterInteractions(work);
};
```

## Conclusion

React Native with Expo provides a powerful, efficient platform for building cross-platform mobile applications. This comprehensive setup guide covers:

**Key Benefits:**
- **Rapid Development**: Hot reloading and instant feedback
- **Cross-Platform**: Single codebase for iOS and Android
- **Rich Ecosystem**: Extensive libraries and community support
- **Easy Deployment**: Streamlined build and distribution process

**What You've Accomplished:**
- Complete development environment setup
- Full-featured task management app with modern patterns
- State management with Redux Toolkit
- Navigation and UI components
- Testing framework and best practices
- Production build and deployment process

**Next Steps:**
- Explore advanced Expo SDK features (camera, location, notifications)
- Implement authentication and user management
- Add offline capabilities with AsyncStorage
- Integrate with backend APIs
- Optimize performance for production
- Set up continuous integration and deployment

React Native with Expo continues to evolve, making mobile development more accessible and efficient. Whether you're building a simple utility app or a complex enterprise application, this setup provides the foundation for successful mobile app development.

The combination of React Native's performance, Expo's developer experience, and the rich ecosystem of libraries makes it an excellent choice for modern mobile development teams looking to build high-quality apps efficiently.