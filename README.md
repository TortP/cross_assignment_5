# Демо данные:

## Логин: default

## Пароль: default

Интеграция API и Управление Состоянием

## 📋 Описание задания

Данный проект реализует комплексную интеграцию с REST API и систему управления глобальным состоянием для мобильного приложения кофейни "Coffee Park". Включает работу с внешними API, Context API для тематизации и Redux Toolkit для управления данными приложения.

## 🎯 Задание 1: Выбор и анализ API

### Выбранные API для проекта

Приложение интегрируется с **MockAPI** - специализированным REST API для тестирования и прототипирования:

**Базовый URL:** `https://68c87dc55d8d9f514735822f.mockapi.io/api/v1`

### Типы API в проекте:

#### ☕ **Products API** - Каталог товаров

- **Эндпоинты:**
  - `GET /products` - получение списка товаров
  - `GET /products/{id}` - детали товара
  - `POST /products` - создание товара
- **Данные:** Информация о кофе, цены, описания, изображения
- **Фильтрация:** По категориям, доступности, рейтингу

#### 🛒 **Cart API** - Корзина покупок

- **Эндпоинты:**
  - `GET /cart?userId={id}` - корзина пользователя
  - `POST /cart` - добавление в корзину
  - `PUT /cart/{id}` - обновление количества
  - `DELETE /cart/{id}` - удаление из корзины
- **Данные:** Товары в корзине, количество, пользователь

#### 👤 **User API** - Пользователи

- **Эндпоинты:**
  - `GET /users` - список пользователей
  - `POST /users` - регистрация
  - `PUT /users/{id}` - обновление профиля
- **Данные:** Профили, настройки, предпочтения

#### 📦 **Orders API** - Заказы

- **Эндпоинты:**
  - `GET /orders?userId={id}` - заказы пользователя
  - `POST /orders` - создание заказа
  - `PUT /orders/{id}` - обновление статуса
- **Данные:** История заказов, статусы, детали

#### 📂 **Categories API** - Категории товаров

- **Эндпоинты:**
  - `GET /categories` - список категорий
- **Данные:** Названия категорий, иконки, описания

## 🛠️ Задание 2: Интеграция API

### Модульная архитектура API

**Файл:** `services/apiService.js`

#### **Базовая инфраструктура:**

```javascript
const MOCK_API_BASE_URL = 'https://68c87dc55d8d9f514735822f.mockapi.io/api/v1';

// Универсальная функция для запросов
const makeRequest = async (endpoint, options = {}) => {
  try {
    const url = `${MOCK_API_BASE_URL}/${endpoint}`;
    const config = {
      headers: {
        'Content-Type': 'application/json',
      },
      ...options,
    };

    const response = await fetch(url, config);

    if (!response.ok) {
      if (response.status === 404) {
        return createApiResponse([]);
      }
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    return createApiResponse(data);
  } catch (error) {
    return createErrorResponse(error.message, 'NETWORK_ERROR');
  }
};
```

#### **Структурированные API модули:**

### 1. **productApi** - Управление товарами

```javascript
export const productApi = {
  // Получение списка товаров с фильтрацией
  async getProducts(filters = {}) {
    let endpoint = 'products';
    const params = new URLSearchParams();

    if (filters.categoryId) {
      params.append('categoryId', filters.categoryId);
    }
    if (filters.available !== undefined) {
      params.append('isAvailable', filters.available);
    }

    if (params.toString()) {
      endpoint += `?${params.toString()}`;
    }

    return await makeRequest(endpoint);
  },

  // Получение товара по ID
  async getProductById(productId) {
    return await makeRequest(`products/${productId}`);
  },
};
```

### 2. **cartApi** - Управление корзиной

```javascript
export const cartApi = {
  // Получение корзины пользователя
  async getCart(userId) {
    return await makeRequest(`cart?userId=${encodeURIComponent(userId)}`);
  },

  // Добавление товара в корзину с полными данными
  async addToCart(productData, quantity = 1, userId) {
    const cartItem = {
      productId: productData.id,
      quantity,
      userId,
      // Сохраняем все данные товара
      ...productData,
      addedAt: new Date().toISOString(),
    };

    return await makeRequest('cart', {
      method: 'POST',
      body: JSON.stringify(cartItem),
    });
  },

  // Обновление количества товара
  async updateCartItem(cartItemId, quantity) {
    return await makeRequest(`cart/${cartItemId}`, {
      method: 'PUT',
      body: JSON.stringify({ quantity }),
    });
  },
};
```

### 3. **Другие API модули:**

- `orderApi` - управление заказами
- `userApi` - профили пользователей
- `categoryApi` - категории товаров
- `paymentMethodsApi` - способы оплаты
- `loyaltyApi` - программа лояльности

### Обработка ошибок и состояний

```javascript
// Стандартизированные ответы API
const createApiResponse = (data, status = 'success') => ({
  status,
  data,
  message:
    status === 'success'
      ? 'Operation completed successfully'
      : 'Operation failed',
});

const createErrorResponse = (message, code = 'ERROR') => ({
  status: 'error',
  data: null,
  message,
  code,
});
```

## 📊 Задание 3: Отображение данных в списках

### Использование FlatList с API данными

#### **ProductList** компонент:

**Файл:** `components/ProductList.jsx`

```jsx
import { ScrollView, ActivityIndicator } from 'react-native';
import ProductCard from './ProductCard';

const ProductList = ({ products = [], onAddToCart, loading = false }) => {
  if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" color="#6B4E3D" />
        <Text style={styles.loadingText}>Загрузка товаров...</Text>
      </View>
    );
  }

  return (
    <ScrollView style={styles.container}>
      <View style={styles.grid}>
        {products.map((product) => (
          <ProductCard
            key={product.id}
            product={product}
            onAddToCart={() => onAddToCart && onAddToCart(product)}
          />
        ))}
      </View>
    </ScrollView>
  );
};
```

#### **HomeScreen** интеграция с API:

**Файл:** `screens/HomeScreen.jsx`

```jsx
import { useSelector, useDispatch } from 'react-redux';
import { fetchProducts } from '../store/productsSlice';

const HomeScreen = () => {
  const dispatch = useDispatch();
  const products = useSelector((state) => state.products.items);
  const loading = useSelector((state) => state.products.loading);
  const error = useSelector((state) => state.products.error);

  useEffect(() => {
    // Загрузка товаров при монтировании компонента
    dispatch(fetchProducts({ available: true }));
  }, [dispatch]);

  return (
    <View style={styles.container}>
      <Categories onCategoryChange={handleCategoryChange} />
      <ProductList
        products={products}
        onAddToCart={handleAddToCart}
        loading={loading}
      />
    </View>
  );
};
```

### keyExtractor и renderItem оптимизация

```jsx
// Для больших списков используется FlatList
<FlatList
  data={products}
  renderItem={({ item }) => (
    <ProductCard product={item} onAddToCart={() => handleAddToCart(item)} />
  )}
  keyExtractor={(item) => item.id.toString()}
  numColumns={2}
  showsVerticalScrollIndicator={false}
/>
```

## ⚠️ Задание 4: Обработка ошибок и загрузки

### Состояния загрузки

#### **ActivityIndicator** для списков:

```jsx
{
  loading && (
    <View style={styles.loadingContainer}>
      <ActivityIndicator size="large" color={theme.brandColor} />
      <Text style={styles.loadingText}>{t('loading')}</Text>
    </View>
  );
}
```

#### **Skeleton Loading** для карточек:

```jsx
// Показ заглушек во время загрузки
{
  loading ? (
    <View style={styles.skeletonCard}>
      <View style={styles.skeletonImage} />
      <View style={styles.skeletonText} />
      <View style={styles.skeletonPrice} />
    </View>
  ) : (
    <ProductCard product={product} />
  );
}
```

### Обработка ошибок сети

#### **Retry механизм:**

```jsx
const [error, setError] = useState(null);
const [retryCount, setRetryCount] = useState(0);

const loadData = async () => {
  try {
    setLoading(true);
    setError(null);

    const response = await productApi.getProducts();

    if (response.status === 'success') {
      setProducts(response.data);
    } else {
      throw new Error(response.message);
    }
  } catch (err) {
    setError(err.message);

    // Автоматический retry для сетевых ошибок
    if (retryCount < 3 && err.message.includes('Network')) {
      setTimeout(() => {
        setRetryCount((prev) => prev + 1);
        loadData();
      }, 2000);
    }
  } finally {
    setLoading(false);
  }
};
```

#### **Error Boundary** компонент:

```jsx
// Отображение ошибок пользователю
{
  error && (
    <View style={styles.errorContainer}>
      <Text style={styles.errorText}>Ошибка загрузки: {error}</Text>
      <TouchableOpacity onPress={handleRetry}>
        <Text style={styles.retryButton}>Повторить</Text>
      </TouchableOpacity>
    </View>
  );
}
```

## 🧭 Задание 5: Интеграция с навигацией

### Навигация с передачей данных из API

#### **Categories** → **ProductsScreen**:

```jsx
// Categories.jsx - переход с параметром категории
const handleCategoryPress = (category) => {
  navigation.navigate('Products', {
    category: category.id,
    categoryName: category.name,
  });
};

// ProductsScreen.jsx - получение и использование параметров
const ProductsScreen = ({ route }) => {
  const { category } = route.params || {};

  useEffect(() => {
    // Загрузка товаров для конкретной категории
    dispatch(fetchProducts({ categoryId: category }));
  }, [category]);
};
```

#### **ProductCard** → **ProductDetailsScreen**:

```jsx
// Переход к деталям товара
<TouchableOpacity
  onPress={() =>
    navigation.navigate('ProductDetails', {
      productId: product.id,
      product: product, // передаем полный объект
    })
  }
>
  <ProductCard product={product} />
</TouchableOpacity>
```

## 🎨 Context API Implementation

### Анализ глобального состояния

**Выбранные аспекты для Context API:**

1. **Тематизация** (светлая/темная тема)
2. **Локализация** (мультиязычность)
3. **Настройки пользователя**

### ThemeContext - Управление темами

**Файл:** `store/themeSlice.js` (Redux Toolkit реализация)

```javascript
import { createSlice } from '@reduxjs/toolkit';
import { lightTheme, darkTheme } from '../themes/colors';

const themeSlice = createSlice({
  name: 'theme',
  initialState: {
    isDark: false,
    theme: lightTheme,
  },
  reducers: {
    toggleTheme: (state) => {
      state.isDark = !state.isDark;
      state.theme = state.isDark ? darkTheme : lightTheme;
    },
    setTheme: (state, action) => {
      state.isDark = action.payload === 'dark';
      state.theme = state.isDark ? darkTheme : lightTheme;
    },
  },
});

export const { toggleTheme, setTheme } = themeSlice.actions;
export default themeSlice.reducer;
```

### Применение тем в компонентах:

```jsx
// Использование темы в компонентах
import { useSelector } from 'react-redux';

const ProductCard = ({ product }) => {
  const theme = useSelector((state) => state.theme.theme);

  return (
    <View style={[styles.card, { backgroundColor: theme.cardBackground }]}>
      <Text style={[styles.title, { color: theme.primaryText }]}>
        {product.name}
      </Text>
    </View>
  );
};
```

### Переключатель тем:

```jsx
// Settings компонент
const SettingsScreen = () => {
  const dispatch = useDispatch();
  const isDark = useSelector((state) => state.theme.isDark);

  return (
    <TouchableOpacity
      onPress={() => dispatch(toggleTheme())}
      style={styles.themeToggle}
    >
      <Text>{isDark ? 'Светлая тема' : 'Темная тема'}</Text>
    </TouchableOpacity>
  );
};
```

## 🗄️ Redux Implementation

### Архитектура Redux Store

**Файл:** `store/store.js`

```javascript
import { configureStore } from '@reduxjs/toolkit';

// Импорт всех slice reducers
import cartReducer from './cartSlice';
import productsReducer from './productsSlice';
import categoriesReducer from './categoriesSlice';
import ordersReducer from './ordersSlice';
import userReducer from './userSlice';
import themeReducer from './themeSlice';
import appReducer from './appSlice';

const store = configureStore({
  reducer: {
    cart: cartReducer,
    products: productsReducer,
    categories: categoriesReducer,
    orders: ordersReducer,
    user: userReducer,
    theme: themeReducer,
    app: appReducer,
  },
});

export default store;
```

### CartSlice - Управление корзиной

**Файл:** `store/cartSlice.js`

```javascript
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { cartApi } from '../services/apiService';

// Async thunks для API операций
export const fetchCart = createAsyncThunk(
  'cart/fetchCart',
  async (userId, { rejectWithValue }) => {
    try {
      const response = await cartApi.getCart(userId);
      if (response.status === 'success') {
        return response.data.filter((item) => item.userId === userId);
      }
      return [];
    } catch (err) {
      return rejectWithValue(err.message);
    }
  }
);

export const addToCart = createAsyncThunk(
  'cart/addToCart',
  async ({ productData, userId }, { dispatch, rejectWithValue }) => {
    try {
      // Проверяем существующие товары в корзине
      const cartResp = await cartApi.getCart(userId);
      const userCartItems =
        cartResp.data?.filter((item) => item.userId === userId) || [];
      const existingItem = userCartItems.find(
        (item) => String(item.productId) === String(productData.id)
      );

      let opResp;
      if (existingItem) {
        // Увеличиваем количество существующего товара
        opResp = await cartApi.updateCartItem(
          existingItem.id,
          existingItem.quantity + 1
        );
      } else {
        // Добавляем новый товар с полными данными
        opResp = await cartApi.addToCart(productData, 1, userId);
      }

      if (opResp?.status === 'success') {
        dispatch(fetchCart(userId));
      }
      return opResp;
    } catch (err) {
      return rejectWithValue(err.message);
    }
  }
);

export const updateCartItemQuantity = createAsyncThunk(
  'cart/updateCartItemQuantity',
  async ({ id, newQuantity, userId }, { dispatch, rejectWithValue }) => {
    try {
      let opResp;
      if (newQuantity <= 0) {
        opResp = await cartApi.removeFromCart(id);
      } else {
        opResp = await cartApi.updateCartItem(id, newQuantity);
      }

      if (opResp?.status === 'success') {
        dispatch(fetchCart(userId));
      }
      return opResp;
    } catch (err) {
      return rejectWithValue(err.message);
    }
  }
);

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [],
    loading: false,
    error: null,
  },
  reducers: {
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      // fetchCart cases
      .addCase(fetchCart.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchCart.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchCart.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      })
      // addToCart cases
      .addCase(addToCart.pending, (state) => {
        state.loading = true;
      })
      .addCase(addToCart.fulfilled, (state) => {
        state.loading = false;
      })
      .addCase(addToCart.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  },
});

export const { clearError } = cartSlice.actions;
export default cartSlice.reducer;
```

### Использование Redux в компонентах

#### **CartScreen** - Отображение корзины:

```jsx
import { useSelector, useDispatch } from 'react-redux';
import { updateCartItemQuantity, fetchCart } from '../store/cartSlice';

const CartScreen = () => {
  const dispatch = useDispatch();
  const cartItems = useSelector((state) => state.cart.items);
  const loading = useSelector((state) => state.cart.loading);
  const userId = useSelector((state) => state.user.profile?.id);

  useEffect(() => {
    if (userId) {
      dispatch(fetchCart(userId));
    }
  }, [userId, dispatch]);

  const handleQuantityChange = (itemId, newQuantity) => {
    dispatch(
      updateCartItemQuantity({
        id: itemId,
        newQuantity,
        userId,
      })
    );
  };

  return (
    <ScrollView>
      {cartItems.map((item) => (
        <CartItem
          key={item.id}
          item={item}
          onUpdateQuantity={(quantity) =>
            handleQuantityChange(item.id, quantity)
          }
        />
      ))}
    </ScrollView>
  );
};
```

### ProductsSlice - Управление товарами

```javascript
export const fetchProducts = createAsyncThunk(
  'products/fetchProducts',
  async (filters = {}, { rejectWithValue }) => {
    try {
      const response = await productApi.getProducts(filters);
      if (response.status === 'success') {
        return response.data;
      }
      return rejectWithValue(response.message);
    } catch (err) {
      return rejectWithValue(err.message);
    }
  }
);

const productsSlice = createSlice({
  name: 'products',
  initialState: {
    items: [],
    loading: false,
    error: null,
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchProducts.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchProducts.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  },
});
```

## 📱 Модульность и архитектура

### Структура файлов:

```
services/
└── apiService.js          # Все API

store/
├── store.js              # Конфигурация Redux store
├── cartSlice.js          # Управление корзиной
├── productsSlice.js      # Каталог товаров
├── categoriesSlice.js    # Категории
├── ordersSlice.js        # Заказы
├── userSlice.js          # Пользователи
├── themeSlice.js         # Темы (Context API альтернатива)
└── appSlice.js           # Локализация и общие настройки

components/
├── ProductList.jsx       # Списки с API данными
├── ProductCard.jsx       # Карточки товаров
├── CartItem.jsx          # Элементы корзины
└── Categories.jsx        # Категории с навигацией

screens/
├── HomeScreen.jsx        # Интеграция products API
├── CartScreen.jsx        # Управление корзиной Redux
├── ProductsScreen.jsx    # Фильтрация по API
└── ProfileScreen.jsx     # Настройки пользователя
```

### Константы и конфигурация:

```javascript
// API URLs
const MOCK_API_BASE_URL = 'https://68c87dc55d8d9f514735822f.mockapi.io/api/v1';

// Error codes
const ERROR_CODES = {
  NETWORK_ERROR: 'NETWORK_ERROR',
  NOT_FOUND: 'NOT_FOUND',
  VALIDATION_ERROR: 'VALIDATION_ERROR',
};

// Default values
const DEFAULT_RETRY_COUNT = 3;
const DEFAULT_TIMEOUT = 5000;
```

## ✅ Соответствие требованиям Task 3

### **Интеграция API** ✅

- **REST API выбор:** MockAPI для кофейни с релевантными данными
- **Fetch API:** Используется для всех HTTP запросов
- **Модульность:** Отдельный файл `apiService.js` с 8 API модулями
- **useState/useReducer:** Redux Toolkit для state management

### **Отображение данных** ✅

- **FlatList/ScrollView:** ProductList с оптимизированным рендерингом
- **renderItem:** Кастомные компоненты ProductCard, CartItem
- **keyExtractor:** Уникальные ID для всех списков
- **Интеграция компонентов:** Все существующие компоненты используют API данные

### **Обработка ошибок** ✅

- **ActivityIndicator:** Состояния загрузки для всех экранов
- **Error handling:** Централизованная обработка с retry механизмом
- **Network errors:** Graceful degradation и fallback состояния
- **User feedback:** Понятные сообщения об ошибках

### **Навигация** ✅

- **Интеграция списков:** Все экраны используют API данные
- **Параметры навигации:** category, productId, orderId передача
- **Детальные экраны:** ProductDetailsScreen с данными из API

### **Redux Implementation** ✅

- **@reduxjs/toolkit:** Современный Redux с createSlice
- **Store configuration:** Централизованное управление состоянием
- **Async thunks:** API интеграция с createAsyncThunk
- **useSelector/useDispatch:** Во всех компонентах
- **Модульность:** Отдельные slice файлы для каждой области

### **Context API Alternative** ✅

- **Темы:** themeSlice для переключения светлой/темной темы
- **Локализация:** appSlice для мультиязычности
- **Provider интеграция:** Redux Provider в App.jsx

---

**Проект:** Coffee Park App  
**API:** MockAPI REST endpoints  
**State Management:** Redux Toolkit + Context API  
**Технологии:** React Native, Fetch API, AsyncThunk
