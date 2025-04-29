# Полное руководство по разработке приложения FirstCall

## Содержание
1. [Введение и обзор проекта](#введение-и-обзор-проекта)
2. [Настройка проекта](#настройка-проекта)
3. [Архитектура приложения](#архитектура-приложения)
4. [Этапы разработки](#этапы-разработки)
5. [Детальная реализация компонентов](#детальная-реализация-компонентов)
6. [Тестирование](#тестирование)
7. [Подготовка к релизу](#подготовка-к-релизу)
8. [Возможные проблемы и их решения](#возможные-проблемы-и-их-решения)
9. [Ресурсы и ссылки](#ресурсы-и-ссылки)

## Введение и обзор проекта

### Описание проекта
FirstCall - мобильное приложение для Android, предназначенное для автоматизации телефонных звонков. Основные функции:
- Авторизация пользователя
- Настройка параметров работы (источники, категории)
- Автоматические звонки по команде от сервера через WebSocket
- Фоновая работа приложения

### Требования к приложению
- Современный дизайн в стиле Material Design
- Поддержка Android 7.0 и выше
- Безопасное хранение данных авторизации
- Стабильное WebSocket соединение с сервером
- Работа приложения поверх других окон

### Ключевые технологии
- Язык программирования: Kotlin
- Архитектура: MVVM (Model-View-ViewModel)
- Сетевые запросы: Retrofit, OkHttp
- Хранение данных: EncryptedSharedPreferences
- UI компоненты: Material Design
- Многопоточность: Coroutines

## Настройка проекта

### Создание проекта в Android Studio

1. Запустите Android Studio
2. Выберите File -> New -> New Project
3. Выберите шаблон "Empty Activity"
4. Заполните настройки проекта:
   - Name: FirstCall
   - Package name: com.firstcall.app
   - Save location: (выберите папку для проекта)
   - Language: Kotlin
   - Minimum SDK: API 24 (Android 7.0)
5. Нажмите "Finish"

### Настройка Gradle зависимостей

Откройте файл `app/build.gradle` и добавьте следующие зависимости:

```gradle
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'kotlin-kapt'
}

android {
    namespace 'com.firstcall.app'
    compileSdk 33

    defaultConfig {
        applicationId "com.firstcall.app"
        minSdk 24 // Android 7.0+
        targetSdk 33
        versionCode 1
        versionName "1.0.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    
    kotlinOptions {
        jvmTarget = '1.8'
    }
    
    buildFeatures {
        viewBinding true
    }
}

dependencies {
    // Android Core
    implementation 'androidx.core:core-ktx:1.9.0'
    implementation 'androidx.appcompat:appcompat:1.6.1'
    
    // Material Design
    implementation 'com.google.android.material:material:1.8.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    
    // ViewModel and LiveData
    implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.5.1'
    implementation 'androidx.lifecycle:lifecycle-livedata-ktx:2.5.1'
    implementation 'androidx.activity:activity-ktx:1.6.1'
    
    // Coroutines
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.4'
    
    // Retrofit для сетевых запросов
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
    
    // OkHttp для WebSocket и перехватчиков
    implementation 'com.squareup.okhttp3:okhttp:4.10.0'
    implementation 'com.squareup.okhttp3:logging-interceptor:4.10.0'
    
    // Gson для работы с JSON
    implementation 'com.google.code.gson:gson:2.10.1'
    
    // Безопасное хранение токенов
    implementation 'androidx.security:security-crypto:1.1.0-alpha06'
    
    // Тестирование
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.5'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
}
```

### Создание структуры пакетов

Создайте следующую структуру пакетов в вашем проекте:

```
com.firstcall.app/
├── data/
│   ├── api/          # API интерфейсы
│   ├── models/       # Модели данных
│   ├── repository/   # Репозитории
│   └── local/        # Локальное хранение данных
├── ui/
│   ├── auth/         # Экран авторизации
│   ├── settings/     # Экран настроек
│   └── service/      # Фоновый сервис
└── utils/            # Утилиты
```

## Архитектура приложения

### Общая архитектура

Приложение реализовано с использованием архитектурного паттерна MVVM (Model-View-ViewModel). 

Схема взаимодействия компонентов:
- **View** (Activity/Fragment) — отображает данные и передаёт действия пользователя в ViewModel
- **ViewModel** — обрабатывает действия, передаёт запросы в Repository и обновляет LiveData
- **Repository** — обрабатывает запросы, получает данные из API или локального хранилища
- **Model** — представляет данные приложения

### Компоненты приложения

1. **Модуль авторизации**
   - LoginActivity — экран ввода логина и пароля
   - LoginViewModel — обработка авторизации
   - AuthRepository — взаимодействие с API авторизации
   - TokenStorage — безопасное хранение токенов

2. **Модуль настроек**
   - SettingsActivity — экран настроек приложения
   - SettingsViewModel — управление настройками
   - SettingsRepository — взаимодействие с API настроек

3. **Модуль телефонных звонков**
   - CallService — фоновый сервис для обработки звонков
   - WebSocketClient — обмен данными с сервером в реальном времени
   - PhoneCallManager — управление телефонными звонками

4. **Вспомогательные компоненты**
   - PermissionManager — работа с разрешениями
   - NetworkModule — настройка Retrofit и OkHttp
   - Result — класс для передачи результатов операций

## Этапы разработки

### Этап 1: Подготовка и настройка проекта (1-2 дня)

#### Задачи:
- Создание проекта в Android Studio
- Настройка зависимостей в Gradle
- Создание базовой структуры проекта
- Добавление необходимых ресурсов

#### Детализация:
1. Создайте проект как описано в разделе "Настройка проекта"
2. Настройте зависимости в `build.gradle`
3. Создайте структуру пакетов
4. Добавьте цвета и строки в ресурсы

#### Ресурсы:
1. Создайте файл `res/values/colors.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="primary">#1976D2</color>
    <color name="primary_dark">#1565C0</color>
    <color name="primary_light">#BBDEFB</color>
    <color name="accent">#FF4081</color>
    <color name="text_primary">#212121</color>
    <color name="text_secondary">#757575</color>
    <color name="background">#FFFFFF</color>
    <color name="error">#F44336</color>
</resources>
```

2. Создайте файл `res/values/strings.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">FirstCall</string>
    
    <!-- Общие строки -->
    <string name="app_logo">Логотип приложения</string>
    <string name="ok">OK</string>
    <string name="cancel">Отмена</string>
    <string name="save">Сохранить</string>
    <string name="error">Ошибка</string>
    <string name="version_format">v%1$s</string>
    
    <!-- Авторизация -->
    <string name="login_hint">Логин</string>
    <string name="password_hint">Пароль</string>
    <string name="login_button">Войти</string>
    <string name="error_login_invalid">Логин не может быть пустым или содержать пробелы</string>
    <string name="error_password_invalid">Пароль не может быть пустым или содержать пробелы</string>
    <string name="error_auth_failed">Неверный логин или пароль</string>
    <string name="error_connection">Сервер временно недоступен</string>
    <string name="error_subscription">Доступ запрещен: подписка истекла</string>
    
    <!-- Настройки -->
    <string name="settings_title">Настройки</string>
    <string name="sources_label">Источники</string>
    <string name="categories_label">Категории</string>
    <string name="logging_label">Логирование</string>
    <string name="autocall_label">Автозвонок</string>
    <string name="telegram_label">Уведомления в Telegram</string>
    <string name="connection_established">Соединение установлено</string>
    <string name="connection_lost">Соединение потеряно</string>
    <string name="settings_saved">Настройки сохранены</string>
    <string name="logout">Выйти</string>
    
    <!-- Сервис -->
    <string name="call_service_name">Сервис звонков</string>
    <string name="call_service_description">Фоновый сервис для обработки звонков</string>
    <string name="notification_title">FirstCall активен</string>
    <string name="notification_text">Сервис работает в фоне</string>
</resources>
```

3. Создайте фон для экрана авторизации в `res/drawable/login_background.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <gradient
        android:angle="135"
        android:endColor="#1976D2"
        android:startColor="#BBDEFB"
        android:type="linear" />
</shape>
```

### Этап 2: Разработка модулей авторизации (2-3 дня)

#### Задачи:
- Создание моделей данных
- Реализация API интерфейсов
- Разработка механизма хранения токенов
- Реализация экрана авторизации

#### Детализация:

1. **Модели данных авторизации**

Создайте файл `data/models/AuthModels.kt`:
```kotlin
package com.firstcall.app.data.models

import com.google.gson.annotations.SerializedName

/**
 * Модель запроса авторизации
 */
data class LoginRequest(
    val login: String,
    val password: String
)

/**
 * Модель ответа на запрос авторизации
 */
data class LoginResponse(
    val code: Int,
    @SerializedName("access_token")
    val accessToken: String,
    @SerializedName("refresh_token")
    val refreshToken: String,
    val sources: List<Source>,
    val categories: List<Category>,
    val logging: Boolean,
    val autocall: Boolean,
    val telegram: Boolean
)

/**
 * Модель ответа на запрос обновления токена
 */
data class TokenResponse(
    @SerializedName("access_token")
    val accessToken: String,
    @SerializedName("refresh_token")
    val refreshToken: String
)

/**
 * Класс для состояний результата операций
 */
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val message: String, val code: Int? = null) : Result<Nothing>()
    object Loading : Result<Nothing>()
}
```

2. **Модели данных настроек**

Создайте файл `data/models/SettingsModels.kt`:
```kotlin
package com.firstcall.app.data.models

/**
 * Модель источника данных
 */
data class Source(
    val id: Int,
    val name: String,
    val enabled: Boolean
)

/**
 * Модель категории
 */
data class Category(
    val id: Int,
    val name: String,
    val enabled: Boolean
)

/**
 * Модель настройки с включенным/выключенным состоянием
 */
data class Setting(
    val enabled: Boolean
)

/**
 * Модель пользовательских настроек
 */
data class UserSettings(
    val sources: List<Source>,
    val categories: List<Category>,
    val logging: Setting,
    val autocall: Setting,
    val telegram: Setting
)
```

3. **API интерфейсы**

Создайте файл `data/api/AuthApi.kt`:
```kotlin
package com.firstcall.app.data.api

import com.firstcall.app.data.models.LoginRequest
import com.firstcall.app.data.models.LoginResponse
import com.firstcall.app.data.models.TokenResponse
import retrofit2.Response
import retrofit2.http.Body
import retrofit2.http.GET
import retrofit2.http.Header
import retrofit2.http.POST

/**
 * Интерфейс для работы с API авторизации
 */
interface AuthApi {
    
    /**
     * Метод для авторизации пользователя
     * @param loginRequest Данные для авторизации (логин и пароль)
     * @return Ответ с токенами и настройками пользователя
     */
    @POST("/api/v1/auth/login")
    suspend fun login(@Body loginRequest: LoginRequest): Response<LoginResponse>
    
    /**
     * Метод для обновления токена доступа
     * @param refreshToken Токен обновления с префиксом Bearer
     * @return Ответ с новыми токенами
     */
    @GET("/api/v1/auth/refresh-token")
    suspend fun refreshToken(@Header("Authorization") refreshToken: String): Response<TokenResponse>
}
```

Создайте файл `data/api/SettingsApi.kt`:
```kotlin
package com.firstcall.app.data.api

import com.firstcall.app.data.models.UserSettings
import retrofit2.Response
import retrofit2.http.Body
import retrofit2.http.GET
import retrofit2.http.Header
import retrofit2.http.PUT

/**
 * Интерфейс для работы с API настроек
 */
interface SettingsApi {
    
    /**
     * Получение текущих настроек пользователя
     * @param token Токен авторизации с префиксом Bearer
     * @return Ответ с настройками пользователя
     */
    @GET("/api/v1/user/settings")
    suspend fun getSettings(@Header("Authorization") token: String): Response<UserSettings>
    
    /**
     * Обновление настроек пользователя
     * @param token Токен авторизации с префиксом Bearer
     * @param settings Новые настройки пользователя
     * @return Ответ с обновленными настройками
     */
    @PUT("/api/v1/user/settings")
    suspend fun updateSettings(
        @Header("Authorization") token: String,
        @Body settings: UserSettings
    ): Response<UserSettings>
}
```

4. **Настройка Retrofit**

Создайте файл `data/api/NetworkModule.kt`:
```kotlin
package com.firstcall.app.data.api

import com.firstcall.app.BuildConfig
import com.google.gson.Gson
import com.google.gson.GsonBuilder
import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory
import java.util.concurrent.TimeUnit

/**
 * Модуль для настройки сетевых компонентов Retrofit и OkHttp
 */
object NetworkModule {
    
    private const val BASE_URL = "https://your-server-url.com" // Замените на адрес вашего сервера
    private const val TIMEOUT = 30L // Таймаут в секундах
    
    /**
     * Создает экземпляр Gson для сериализации/десериализации JSON
     */
    private fun provideGson(): Gson {
        return GsonBuilder()
            .setLenient()
            .create()
    }
    
    /**
     * Создает OkHttpClient с логированием для отладки
     */
    private fun provideOkHttpClient(): OkHttpClient {
        val loggingInterceptor = HttpLoggingInterceptor().apply {
            level = if (BuildConfig.DEBUG) {
                HttpLoggingInterceptor.Level.BODY
            } else {
                HttpLoggingInterceptor.Level.NONE
            }
        }
        
        return OkHttpClient.Builder()
            .addInterceptor(loggingInterceptor)
            .connectTimeout(TIMEOUT, TimeUnit.SECONDS)
            .readTimeout(TIMEOUT, TimeUnit.SECONDS)
            .writeTimeout(TIMEOUT, TimeUnit.SECONDS)
            .build()
    }
    
    /**
     * Создает Retrofit для API запросов
     */
    private fun provideRetrofit(client: OkHttpClient, gson: Gson): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(client)
            .addConverterFactory(GsonConverterFactory.create(gson))
            .build()
    }
    
    /**
     * Предоставляет интерфейс AuthApi
     */
    fun provideAuthApi(): AuthApi {
        val gson = provideGson()
        val client = provideOkHttpClient()
        val retrofit = provideRetrofit(client, gson)
        return retrofit.create(AuthApi::class.java)
    }
    
    /**
     * Предоставляет интерфейс SettingsApi
     */
    fun provideSettingsApi(): SettingsApi {
        val gson = provideGson()
        val client = provideOkHttpClient()
        val retrofit = provideRetrofit(client, gson)
        return retrofit.create(SettingsApi::class.java)
    }
}
```

5. **Хранение токенов**

Создайте файл `data/local/TokenStorage.kt`:
```kotlin
package com.firstcall.app.data.local

import android.content.Context
import androidx.security.crypto.EncryptedSharedPreferences
import androidx.security.crypto.MasterKeys

/**
 * Класс для безопасного хранения токенов авторизации
 * Использует EncryptedSharedPreferences для шифрования данных
 */
class TokenStorage(context: Context) {
    
    private val masterKeyAlias = MasterKeys.getOrCreate(MasterKeys.AES256_GCM_SPEC)
    
    private val sharedPreferences = EncryptedSharedPreferences.create(
        "auth_tokens",
        masterKeyAlias,
        context,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )
    
    /**
     * Сохраняет токены авторизации
     * @param accessToken Короткоживущий токен доступа
     * @param refreshToken Токен обновления
     */
    fun saveTokens(accessToken: String, refreshToken: String) {
        sharedPreferences.edit()
            .putString(KEY_ACCESS_TOKEN, accessToken)
            .putString(KEY_REFRESH_TOKEN, refreshToken)
            .apply()
    }
    
    /**
     * Получает токен доступа
     * @return Токен доступа или null, если не сохранен
     */
    fun getAccessToken(): String? = sharedPreferences.getString(KEY_ACCESS_TOKEN, null)
    
    /**
     * Получает токен обновления
     * @return Токен обновления или null, если не сохранен
     */
    fun getRefreshToken(): String? = sharedPreferences.getString(KEY_REFRESH_TOKEN, null)
    
    /**
     * Обновляет только токен доступа
     * @param accessToken Новый токен доступа
     */
    fun updateAccessToken(accessToken: String) {
        sharedPreferences.edit()
            .putString(KEY_ACCESS_TOKEN, accessToken)
            .apply()
    }
    
    /**
     * Проверяет наличие токенов авторизации
     * @return true, если токены сохранены
     */
    fun hasTokens(): Boolean = getAccessToken() != null && getRefreshToken() != null
    
    /**
     * Очищает все токены (выход из системы)
     */
    fun clearTokens() {
        sharedPreferences.edit().clear().apply()
    }
    
    companion object {
        private const val KEY_ACCESS_TOKEN = "access_token"
        private const val KEY_REFRESH_TOKEN = "refresh_token"
    }
}
```

6. **Репозиторий авторизации**

Создайте файл `data/repository/AuthRepository.kt`:
```kotlin
package com.firstcall.app.data.repository

import android.content.Context
import com.firstcall.app.R
import com.firstcall.app.data.api.AuthApi
import com.firstcall.app.data.api.NetworkModule
import com.firstcall.app.data.local.TokenStorage
import com.firstcall.app.data.models.LoginRequest
import com.firstcall.app.data.models.LoginResponse
import com.firstcall.app.data.models.Result
import com.firstcall.app.data.models.TokenResponse
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import java.io.IOException

/**
 * Репозиторий для работы с авторизацией
 */
class AuthRepository(
    private val context: Context,
    private val api: AuthApi = NetworkModule.provideAuthApi(),
    private val tokenStorage: TokenStorage = TokenStorage(context)
) {
    
    /**
     * Проверка авторизации пользователя
     * @return true, если пользователь авторизован
     */
    fun isUserLoggedIn(): Boolean {
        return tokenStorage.hasTokens()
    }
    
    /**
     * Авторизация пользователя
     * @param login Логин пользователя
     * @param password Пароль пользователя
     * @return Результат операции с данными пользователя или ошибкой
     */
    suspend fun login(login: String, password: String): Result<LoginResponse> = withContext(Dispatchers.IO) {
        try {
            val response = api.login(LoginRequest(login, password))
            
            if (response.isSuccessful) {
                val loginResponse = response.body()
                if (loginResponse != null) {
                    // Сохранение токенов в безопасное хранилище
                    tokenStorage.saveTokens(loginResponse.accessToken, loginResponse.refreshToken)
                    return@withContext Result.Success(loginResponse)
                } else {
                    return@withContext Result.Error(context.getString(R.string.error_connection))
                }
            } else {
                // Обработка ошибок от сервера
                return@withContext when (response.code()) {
                    401 -> Result.Error(context.getString(R.string.error_auth_failed))
                    403 -> Result.Error(context.getString(R.string.error_subscription))
                    422 -> Result.Error("Ошибка валидации")
                    else -> Result.Error(context.getString(R.string.error_connection))
                }
            }
        } catch (e: IOException) {
            // Ошибка сети
            return@withContext Result.Error(context.getString(R.string.error_connection))
        } catch (e: Exception) {
            // Другие ошибки
            return@withContext Result.Error(e.message ?: context.getString(R.string.error))
        }
    }
    
    /**
     * Обновление токена доступа
     * @return Результат операции с новыми токенами или ошибкой
     */
    suspend fun refreshToken(): Result<TokenResponse> = withContext(Dispatchers.IO) {
        try {
            val refreshToken = tokenStorage.getRefreshToken()
                ?: return@withContext Result.Error("Нет refresh токена")
            
            val response = api.refreshToken("Bearer $refreshToken")
            
            if (response.isSuccessful) {
                val tokenResponse = response.body()
                if (tokenResponse != null) {
                    // Сохранение новых токенов
                    tokenStorage.saveTokens(tokenResponse.accessToken, tokenResponse.refreshToken)
                    return@withContext Result.Success(tokenResponse)
                } else {
                    return@withContext Result.Error(context.getString(R.string.error_connection))
                }
            } else {
                // Обработка ошибок обновления токена
                if (response.code() == 401) {
                    // Токен недействителен, выход из системы
                    tokenStorage.clearTokens()
                    return@withContext Result.Error("Требуется повторная авторизация", 401)
                } else {
                    return@withContext Result.Error(context.getString(R.string.error_connection))
                }
            }
        } catch (e: IOException) {
            return@withContext Result.Error(context.getString(R.string.error_connection))
        } catch (e: Exception) {
            return@withContext Result.Error(e.message ?: context.getString(R.string.error))
        }
    }
    
    /**
     * Выход из системы (очистка токенов)
     */
    fun logout() {
        tokenStorage.clearTokens()
    }
}
```

7. **Экран авторизации**

Создайте файл `res/layout/activity_login.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/login_background"
    android:padding="16dp"
    tools:context=".ui.auth.LoginActivity">

    <ImageView
        android:id="@+id/logoImageView"
        android:layout_width="120dp"
        android:layout_height="120dp"
        android:layout_marginTop="48dp"
        android:contentDescription="@string/app_logo"
        android:src="@drawable/ic_app_logo"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/appNameTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:text="@string/app_name"
        android:textColor="@color/primary"
        android:textSize="24sp"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/logoImageView" />

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/loginInputLayout"
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="48dp"
        android:hint="@string/login_hint"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/appNameTextView">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/loginField"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="text"
            android:maxLines="1" />
    </com.google.android.material.textfield.TextInputLayout>

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/passwordInputLayout"
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:hint="@string/password_hint"
        app:endIconMode="password_toggle"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/loginInputLayout">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/passwordField"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textPassword"
            android:maxLines="1" />
    </com.google.android.material.textfield.TextInputLayout>

    <com.google.android.material.button.MaterialButton
        android:id="@+id/loginButton"
        android:layout_width="match_parent"
        android:layout_height="56dp"
        android:layout_marginTop="32dp"
        android:text="@string/login_button"
        android:textAllCaps="false"
        android:textSize="16sp"
        app:cornerRadius="8dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/passwordInputLayout" />

    <ProgressBar
        android:id="@+id/progressBar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:visibility="gone"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/loginButton" />

    <TextView
        android:id="@+id/versionText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="16dp"
        android:text="v1.0.0"
        android:textColor="@color/text_secondary"
        android:textSize="12sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

8. **ViewModel для экрана авторизации**

Создайте файл `ui/auth/LoginViewModel.kt`:
```kotlin
package com.firstcall.app.ui.auth

import android.content.Context
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import androidx.lifecycle.viewModelScope
import com.firstcall.app.data.models.LoginResponse
import com.firstcall.app.data.models.Result
import com.firstcall.app.data.repository.AuthRepository
import kotlinx.coroutines.launch

/**
 * ViewModel для экрана авторизации
 */
class LoginViewModel(private val repository: AuthRepository) : ViewModel() {
    
    // LiveData для отслеживания результата авторизации
    private val _loginResult = MutableLiveData<Result<LoginResponse>>()
    val loginResult: LiveData<Result<LoginResponse>> = _loginResult
    
    /**
     * Проверка авторизации пользователя
     * @return true, если пользователь авторизован
     */
    fun isUserLoggedIn(): Boolean {
        return repository.isUserLoggedIn()
    }
    
    /**
     * Авторизация пользователя
     * @param login Логин пользователя
     * @param password Пароль пользователя
     */
    fun login(login: String, password: String) {
        // Установка состояния загрузки
        _loginResult.value = Result.Loading
        
        // Запуск корутины для асинхронной операции
        viewModelScope.launch {
            val result = repository.login(login, password)
            _loginResult.postValue(result)
        }
    }
}

/**
 * Фабрика для создания LoginViewModel с нужными зависимостями
 */
class LoginViewModelFactory(private val context: Context) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(LoginViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return LoginViewModel(AuthRepository(context)) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

9. **Активность для экрана авторизации**

Создайте файл `ui/auth/LoginActivity.kt`:
```kotlin
package com.firstcall.app.ui.auth

import android.content.Intent
import android.os.Bundle
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.isVisible
import com.firstcall.app.BuildConfig
import com.firstcall.app.R
import com.firstcall.app.data.models.Result
import com.firstcall.app.databinding.ActivityLoginBinding
import com.firstcall.app.ui.settings.SettingsActivity
import com.google.android.material.snackbar.Snackbar

/**
 * Экран авторизации пользователя в приложении
 */
class LoginActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityLoginBinding
    private val viewModel: LoginViewModel by viewModels { LoginViewModelFactory(applicationContext) }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Проверка наличия сохраненных токенов
        if (viewModel.isUserLoggedIn()) {
            navigateToSettings()
            return
        }
        
        binding = ActivityLoginBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        setupUI()
        observeViewModel()
    }
    
    private fun setupUI() {
        // Установка версии приложения
        binding.versionText.text = getString(R.string.version_format, BuildConfig.VERSION_NAME)
        
        // Настройка обработчика нажатия на кнопку входа
        binding.loginButton.setOnClickListener {
            val login = binding.loginField.text.toString()
            val password = binding.passwordField.text.toString()
            
            if (validateInputs(login, password)) {
                viewModel.login(login, password)
            }
        }
    }
    
    /**
     * Валидация полей ввода логина и пароля
     */
    private fun validateInputs(login: String, password: String): Boolean {
        var isValid = true
        
        // Проверка логина
        if (login.isBlank() || login.contains(" ")) {
            binding.loginInputLayout.error = getString(R.string.error_login_invalid)
            isValid = false
        } else {
            binding.loginInputLayout.error = null
        }
        
        // Проверка пароля
        if (password.isBlank() || password.contains(" ")) {
            binding.passwordInputLayout.error = getString(R.string.error_password_invalid)
            isValid = false
        } else {
            binding.passwordInputLayout.error = null
        }
        
        return isValid
    }
    
    /**
     * Наблюдение за изменениями состояния viewModel
     */
    private fun observeViewModel() {
        viewModel.loginResult.observe(this) { result ->
            when (result) {
                is Result.Loading -> {
                    binding.progressBar.isVisible = true
                    binding.loginButton.isEnabled = false
                }
                is Result.Success -> {
                    binding.progressBar.isVisible = false
                    binding.loginButton.isEnabled = true
                    
                    // Переход на экран настроек
                    navigateToSettings()
                }
                is Result.Error -> {
                    binding.progressBar.isVisible = false
                    binding.loginButton.isEnabled = true
                    
                    // Отображение ошибки
                    Snackbar.make(
                        binding.root,
                        result.message,
                        Snackbar.LENGTH_LONG
                    ).show()
                }
            }
        }
    }
    
    /**
     * Переход на экран настроек
     */
    private fun navigateToSettings() {
        startActivity(Intent(this, SettingsActivity::class.java))
        finish()
    }
}
```

### Этап 3: Разработка модуля настроек (3-4 дня)

#### Задачи:
- Создание репозитория настроек
- Разработка экрана настроек
- Реализация логики работы с настройками

#### Детализация:

1. **Репозиторий настроек**

Создайте файл `data/repository/SettingsRepository.kt`:
```kotlin
package com.firstcall.app.data.repository

import android.content.Context
import com.firstcall.app.R
import com.firstcall.app.data.api.NetworkModule
import com.firstcall.app.data.api.SettingsApi
import com.firstcall.app.data.local.TokenStorage
import com.firstcall.app.data.models.Result
import com.firstcall.app.data.models.UserSettings
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import java.io.IOException

/**
 * Репозиторий для работы с настройками пользователя
 */
class SettingsRepository(
    private val context: Context,
    private val api: SettingsApi = NetworkModule.provideSettingsApi(),
    private val tokenStorage: TokenStorage = TokenStorage(context),
    private val authRepository: AuthRepository = AuthRepository(context)
) {
    
    /**
     * Получение настроек пользователя
     * @return Результат операции с настройками или ошибкой
     */
    suspend fun getSettings(): Result<UserSettings> = withContext(Dispatchers.IO) {
        try {
            val token = tokenStorage.getAccessToken() 
                ?: return@withContext Result.Error("Токен не найден", 401)
            
            val response = api.getSettings("Bearer $token")
            
            if (response.isSuccessful) {
                val settings = response.body()
                if (settings != null) {
                    return@withContext Result.Success(settings)
                } else {
                    return@withContext Result.Error(context.getString(R.string.error_connection))
                }
            } else {
                // Обработка ошибок от сервера
                return@withContext when (response.code()) {
                    401 -> {
                        // Токен истек, пробуем обновить
                        handleTokenRefresh { getSettings() }
                    }
                    403 -> {
                        // Подписка истекла, выход из системы
                        tokenStorage.clearTokens()
                        Result.Error(context.getString(R.string.error_subscription), 403)
                    }
                    else -> Result.Error(context.getString(R.string.error_connection))
                }
            }
        } catch (e: IOException) {
            return@withContext Result.Error(context.getString(R.string.error_connection))
        } catch (e: Exception) {
            return@withContext Result.Error(e.message ?: context.getString(R.string.error))
        }
    }
    
    /**
     * Обновление настроек пользователя
     * @param settings Новые настройки пользователя
     * @return Результат операции с обновленными настройками или ошибкой
     */
    suspend fun updateSettings(settings: UserSettings): Result<UserSettings> = withContext(Dispatchers.IO) {
        try {
            val token = tokenStorage.getAccessToken()
                ?: return@withContext Result.Error("Токен не найден", 401)
            
            val response = api.updateSettings("Bearer $token", settings)
            
            if (response.isSuccessful) {
                val updatedSettings = response.body()
                if (updatedSettings != null) {
                    return@withContext Result.Success(updatedSettings)
                } else {
                    return@withContext Result.Error(context.getString(R.string.error_connection))
                }
            } else {
                // Обработка ошибок от сервера
                return@withContext when (response.code()) {
                    401 -> {
                        // Токен истек, пробуем обновить
                        handleTokenRefresh { updateSettings(settings) }
                    }
                    403 -> {
                        // Подписка истекла, выход из системы
                        tokenStorage.clearTokens()
                        Result.Error(context.getString(R.string.error_subscription), 403)
                    }
                    else -> Result.Error(context.getString(R.string.error_connection))
                }
            }
        } catch (e: IOException) {
            return@withContext Result.Error(context.getString(R.string.error_connection))
        } catch (e: Exception) {
            return@withContext Result.Error(e.message ?: context.getString(R.string.error))
        }
    }
    
    /**
     * Обработка обновления токена и повторного выполнения операции
     * @param operation Операция, которую нужно повторить после обновления токена
     */
    private suspend fun <T> handleTokenRefresh(operation: suspend () -> Result<T>): Result<T> {
        // Пытаемся обновить токен
        val refreshResult = authRepository.refreshToken()
        
        return if (refreshResult is Result.Success) {
            // Токен успешно обновлен, повторяем операцию
            operation()
        } else {
            // Не удалось обновить токен, возвращаем ошибку
            Result.Error("Требуется повторная авторизация", 401)
        }
    }
}
```

2. **Макет элемента источника**

Создайте файл `res/layout/item_source.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="8dp">

    <TextView
        android:id="@+id/sourceName"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginEnd="16dp"
        android:textColor="@color/text_primary"
        android:textSize="16sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/sourceSwitch"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.switchmaterial.SwitchMaterial
        android:id="@+id/sourceSwitch"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

3. **Макет элемента категории**

Создайте файл `res/layout/item_category.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="8dp">

    <TextView
        android:id="@+id/categoryName"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginEnd="16dp"
        android:textColor="@color/text_primary"
        android:textSize="16sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/categorySwitch"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.switchmaterial.SwitchMaterial
        android:id="@+id/categorySwitch"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

4. **Макет экрана настроек**

Создайте файл `res/layout/activity_settings.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp"
    tools:context=".ui.settings.SettingsActivity">

    <TextView
        android:id="@+id/settingsTitle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/settings_title"
        android:textColor="@color/text_primary"
        android:textSize="20sp"
        android:textStyle="bold"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <ScrollView
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_marginTop="16dp"
        android:layout_marginBottom="16dp"
        app:layout_constraintBottom_toTopOf="@+id/saveButton"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/settingsTitle">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="16dp"
                app:cardCornerRadius="8dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="@string/sources_label"
                        android:textColor="@color/text_primary"
                        android:textSize="18sp"
                        android:textStyle="bold" />

                    <androidx.recyclerview.widget.RecyclerView
                        android:id="@+id/sourcesRecyclerView"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="8dp"
                        android:nestedScrollingEnabled="false"
                        tools:itemCount="3"
                        tools:listitem="@layout/item_source" />
                </LinearLayout>
            </com.google.android.material.card.MaterialCardView>

            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="16dp"
                app:cardCornerRadius="8dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="@string/categories_label"
                        android:textColor="@color/text_primary"
                        android:textSize="18sp"
                        android:textStyle="bold" />

                    <androidx.recyclerview.widget.RecyclerView
                        android:id="@+id/categoriesRecyclerView"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="8dp"
                        android:nestedScrollingEnabled="false"
                        tools:itemCount="3"
                        tools:listitem="@layout/item_category" />
                </LinearLayout>
            </com.google.android.material.card.MaterialCardView>

            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="16dp"
                app:cardCornerRadius="8dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="@string/settings_title"
                        android:textColor="@color/text_primary"
                        android:textSize="18sp"
                        android:textStyle="bold" />

                    <androidx.constraintlayout.widget.ConstraintLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="8dp">

                        <TextView
                            android:id="@+id/loggingLabel"
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_marginEnd="16dp"
                            android:text="@string/logging_label"
                            android:textColor="@color/text_primary"
                            android:textSize="16sp"
                            app:layout_constraintBottom_toBottomOf="parent"
                            app:layout_constraintEnd_toStartOf="@+id/loggingSwitch"
                            app:layout_constraintStart_toStartOf="parent"
                            app:layout_constraintTop_toTopOf="parent" />

                        <com.google.android.material.switchmaterial.SwitchMaterial
                            android:id="@+id/loggingSwitch"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            app:layout_constraintBottom_toBottomOf="parent"
                            app:layout_constraintEnd_toEndOf="parent"
                            app:layout_constraintTop_toTopOf="parent" />
                    </androidx.constraintlayout.widget.ConstraintLayout>

                    <androidx.constraintlayout.widget.ConstraintLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="16dp">

                        <TextView
                            android:id="@+id/autocallLabel"
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_marginEnd="16dp"
                            android:text="@string/autocall_label"
                            android:textColor="@color/text_primary"
                            android:textSize="16sp"
                            app:layout_constraintBottom_toBottomOf="parent"
                            app:layout_constraintEnd_toStartOf="@+id/autocallSwitch"
                            app:layout_constraintStart_toStartOf="parent"
                            app:layout_constraintTop_toTopOf="parent" />

                        <com.google.android.material.switchmaterial.SwitchMaterial
                            android:id="@+id/autocallSwitch"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            app:layout_constraintBottom_toBottomOf="parent"
                            app:layout_constraintEnd_toEndOf="parent"
                            app:layout_constraintTop_toTopOf="parent" />
                    </androidx.constraintlayout.widget.ConstraintLayout>

                    <androidx.constraintlayout.widget.ConstraintLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="16dp">

                        <TextView
                            android:id="@+id/telegramLabel"
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_marginEnd="16dp"
                            android:text="@string/telegram_label"
                            android:textColor="@color/text_primary"
                            android:textSize="16sp"
                            app:layout_constraintBottom_toBottomOf="parent"
                            app:layout_constraintEnd_toStartOf="@+id/telegramSwitch"
                            app:layout_constraintStart_toStartOf="parent"
                            app:layout_constraintTop_toTopOf="parent" />

                        <com.google.android.material.switchmaterial.SwitchMaterial
                            android:id="@+id/telegramSwitch"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            app:layout_constraintBottom_toBottomOf="parent"
                            app:layout_constraintEnd_toEndOf="parent"
                            app:layout_constraintTop_toTopOf="parent" />
                    </androidx.constraintlayout.widget.ConstraintLayout>
                </LinearLayout>
            </com.google.android.material.card.MaterialCardView>
        </LinearLayout>
    </ScrollView>

    <com.google.android.material.button.MaterialButton
        android:id="@+id/saveButton"
        android:layout_width="0dp"
        android:layout_height="56dp"
        android:layout_marginEnd="8dp"
        android:text="@string/save"
        android:textAllCaps="false"
        android:textSize="16sp"
        app:cornerRadius="8dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/logoutButton"
        app:layout_constraintStart_toStartOf="parent" />

    <com.google.android.material.button.MaterialButton
        android:id="@+id/logoutButton"
        style="@style/Widget.MaterialComponents.Button.OutlinedButton"
        android:layout_width="wrap_content"
        android:layout_height="56dp"
        android:text="@string/logout"
        android:textAllCaps="false"
        android:textSize="16sp"
        app:cornerRadius="8dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <ProgressBar
        android:id="@+id/progressBar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:visibility="gone"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

5. **Адаптер для источников**

Создайте файл `ui/settings/SourceAdapter.kt`:
```kotlin
package com.firstcall.app.ui.settings

import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.DiffUtil
import androidx.recyclerview.widget.ListAdapter
import androidx.recyclerview.widget.RecyclerView
import com.firstcall.app.data.models.Source
import com.firstcall.app.databinding.ItemSourceBinding

/**
 * Адаптер для отображения списка источников в RecyclerView
 */
class SourceAdapter(private val onSourceToggled: (Source, Boolean) -> Unit) :
    ListAdapter<Source, SourceAdapter.SourceViewHolder>(SourceDiffCallback()) {
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): SourceViewHolder {
        val binding = ItemSourceBinding.inflate(
            LayoutInflater.from(parent.context),
            parent,
            false
        )
        return SourceViewHolder(binding)
    }
    
    override fun onBindViewHolder(holder: SourceViewHolder, position: Int) {
        val source = getItem(position)
        holder.bind(source)
    }
    
    inner class SourceViewHolder(private val binding: ItemSourceBinding) :
        RecyclerView.ViewHolder(binding.root) {
        
        fun bind(source: Source) {
            binding.sourceName.text = source.name
            binding.sourceSwitch.isChecked = source.enabled
            
            // Установка слушателя изменения состояния переключателя
            binding.sourceSwitch.setOnCheckedChangeListener { _, isChecked ->
                onSourceToggled(source, isChecked)
            }
        }
    }
    
    /**
     * DiffCallback для оптимизации обновления списка
     */
    class SourceDiffCallback : DiffUtil.ItemCallback<Source>() {
        override fun areItemsTheSame(oldItem: Source, newItem: Source): Boolean {
            return oldItem.id == newItem.id
        }
        
        override fun areContentsTheSame(oldItem: Source, newItem: Source): Boolean {
            return oldItem == newItem
        }
    }
    
    /**
     * Обновляет состояние источника в списке
     */
    fun updateSourceState(sourceId: Int, enabled: Boolean) {
        val currentList = currentList.toMutableList()
        val index = currentList.indexOfFirst { it.id == sourceId }
        
        if (index != -1) {
            val oldSource = currentList[index]
            val newSource = oldSource.copy(enabled = enabled)
            currentList[index] = newSource
            submitList(currentList)
        }
    }
}
```

6. **Адаптер для категорий**

Создайте файл `ui/settings/CategoryAdapter.kt`:
```kotlin
package com.firstcall.app.ui.settings

import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.DiffUtil
import androidx.recyclerview.widget.ListAdapter
import androidx.recyclerview.widget.RecyclerView
import com.firstcall.app.data.models.Category
import com.firstcall.app.databinding.ItemCategoryBinding

/**
 * Адаптер для отображения списка категорий в RecyclerView
 */
class CategoryAdapter(private val onCategoryToggled: (Category, Boolean) -> Unit) :
    ListAdapter<Category, CategoryAdapter.CategoryViewHolder>(CategoryDiffCallback()) {
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): CategoryViewHolder {
        val binding = ItemCategoryBinding.inflate(
            LayoutInflater.from(parent.context),
            parent,
            false
        )
        return CategoryViewHolder(binding)
    }
    
    override fun onBindViewHolder(holder: CategoryViewHolder, position: Int) {
        val category = getItem(position)
        holder.bind(category)
    }
    
    inner class CategoryViewHolder(private val binding: ItemCategoryBinding) :
        RecyclerView.ViewHolder(binding.root) {
        
        fun bind(category: Category) {
            binding.categoryName.text = category.name
            binding.categorySwitch.isChecked = category.enabled
            
            // Установка слушателя изменения состояния переключателя
            binding.categorySwitch.setOnCheckedChangeListener { _, isChecked ->
                onCategoryToggled(category, isChecked)
            }
        }
    }
    
    /**
     * DiffCallback для оптимизации обновления списка
     */
    class CategoryDiffCallback : DiffUtil.ItemCallback<Category>() {
        override fun areItemsTheSame(oldItem: Category, newItem: Category): Boolean {
            return oldItem.id == newItem.id
        }
        
        override fun areContentsTheSame(oldItem: Category, newItem: Category): Boolean {
            return oldItem == newItem
        }
    }
    
    /**
     * Обновляет состояние категории в списке
     */
    fun updateCategoryState(categoryId: Int, enabled: Boolean) {
        val currentList = currentList.toMutableList()
        val index = currentList.indexOfFirst { it.id == categoryId }
        
        if (index != -1) {
            val oldCategory = currentList[index]
            val newCategory = oldCategory.copy(enabled = enabled)
            currentList[index] = newCategory
            submitList(currentList)
        }
    }
}
```

7. **ViewModel для экрана настроек**

Создайте файл `ui/settings/SettingsViewModel.kt`:
```kotlin
package com.firstcall.app.ui.settings

import android.content.Context
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import androidx.lifecycle.viewModelScope
import com.firstcall.app.data.models.Category
import com.firstcall.app.data.models.Result
import com.firstcall.app.data.models.Setting
import com.firstcall.app.data.models.Source
import com.firstcall.app.data.models.UserSettings
import com.firstcall.app.data.repository.AuthRepository
import com.firstcall.app.data.repository.SettingsRepository
import kotlinx.coroutines.launch

/**
 * ViewModel для экрана настроек
 */
class SettingsViewModel(
    private val settingsRepository: SettingsRepository,
    private val authRepository: AuthRepository
) : ViewModel() {
    
    // LiveData для отслеживания загрузки настроек
    private val _settingsResult = MutableLiveData<Result<UserSettings>>()
    val settingsResult: LiveData<Result<UserSettings>> = _settingsResult
    
    // LiveData для обновления настроек
    private val _updateResult = MutableLiveData<Result<UserSettings>>()
    val updateResult: LiveData<Result<UserSettings>> = _updateResult
    
    // Текущие настройки пользователя
    private var currentSettings: UserSettings? = null
    
    // Мутабельные списки для отслеживания изменений
    private val _sources = MutableLiveData<List<Source>>()
    val sources: LiveData<List<Source>> = _sources
    
    private val _categories = MutableLiveData<List<Category>>()
    val categories: LiveData<List<Category>> = _categories
    
    private val _loggingEnabled = MutableLiveData<Boolean>()
    val loggingEnabled: LiveData<Boolean> = _loggingEnabled
    
    private val _autocallEnabled = MutableLiveData<Boolean>()
    val autocallEnabled: LiveData<Boolean> = _autocallEnabled
    
    private val _telegramEnabled = MutableLiveData<Boolean>()
    val telegramEnabled: LiveData<Boolean> = _telegramEnabled
    
    /**
     * Загрузка настроек пользователя
     */
    fun loadSettings() {
        _settingsResult.value = Result.Loading
        
        viewModelScope.launch {
            val result = settingsRepository.getSettings()
            
            if (result is Result.Success) {
                // Сохранение текущих настроек
                currentSettings = result.data
                
                // Обновление LiveData с настройками
                _sources.postValue(result.data.sources)
                _categories.postValue(result.data.categories)
                _loggingEnabled.postValue(result.data.logging.enabled)
                _autocallEnabled.postValue(result.data.autocall.enabled)
                _telegramEnabled.postValue(result.data.telegram.enabled)
            }
            
            _settingsResult.postValue(result)
        }
    }
    
    /**
     * Обновление состояния источника
     */
    fun updateSourceState(sourceId: Int, enabled: Boolean) {
        val currentList = _sources.value?.toMutableList() ?: return
        val index = currentList.indexOfFirst { it.id == sourceId }
        
        if (index != -1) {
            val oldSource = currentList[index]
            val newSource = oldSource.copy(enabled = enabled)
            currentList[index] = newSource
            _sources.value = currentList
        }
    }
    
    /**
     * Обновление состояния категории
     */
    fun updateCategoryState(categoryId: Int, enabled: Boolean) {
        val currentList = _categories.value?.toMutableList() ?: return
        val index = currentList.indexOfFirst { it.id == categoryId }
        
        if (index != -1) {
            val oldCategory = currentList[index]
            val newCategory = oldCategory.copy(enabled = enabled)
            currentList[index] = newCategory
            _categories.value = currentList
        }
    }
    
    /**
     * Обновление состояния логирования
     */
    fun updateLoggingState(enabled: Boolean) {
        _loggingEnabled.value = enabled
    }
    
    /**
     * Обновление состояния автозвонка
     */
    fun updateAutocallState(enabled: Boolean) {
        _autocallEnabled.value = enabled
    }
    
    /**
     * Обновление состояния Telegram уведомлений
     */
    fun updateTelegramState(enabled: Boolean) {
        _telegramEnabled.value = enabled
    }
    
    /**
     * Сохранение всех настроек на сервере
     */
    fun saveSettings() {
        val sources = _sources.value ?: return
        val categories = _categories.value ?: return
        val loggingEnabled = _loggingEnabled.value ?: false
        val autocallEnabled = _autocallEnabled.value ?: false
        val telegramEnabled = _telegramEnabled.value ?: false
        
        val settings = UserSettings(
            sources = sources,
            categories = categories,
            logging = Setting(loggingEnabled),
            autocall = Setting(autocallEnabled),
            telegram = Setting(telegramEnabled)
        )
        
        _updateResult.value = Result.Loading
        
        viewModelScope.launch {
            val result = settingsRepository.updateSettings(settings)
            
            if (result is Result.Success) {
                // Обновление текущих настроек
                currentSettings = result.data
                
                // Обновление LiveData
                _sources.postValue(result.data.sources)
                _categories.postValue(result.data.categories)
                _loggingEnabled.postValue(result.data.logging.enabled)
                _autocallEnabled.postValue(result.data.autocall.enabled)
                _telegramEnabled.postValue(result.data.telegram.enabled)
            }
            
            _updateResult.postValue(result)
        }
    }
    
    /**
     * Выход из системы
     */
    fun logout() {
        authRepository.logout()
    }
}

/**
 * Фабрика для создания SettingsViewModel с нужными зависимостями
 */
class SettingsViewModelFactory(private val context: Context) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(SettingsViewModel::class.java)) {
            val settingsRepository = SettingsRepository(context)
            val authRepository = AuthRepository(context)
            
            @Suppress("UNCHECKED_CAST")
            return SettingsViewModel(settingsRepository, authRepository) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

8. **Активность экрана настроек**

Создайте файл `ui/settings/SettingsActivity.kt`:
```kotlin
package com.firstcall.app.ui.settings

import android.content.Intent
import android.os.Bundle
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.isVisible
import androidx.recyclerview.widget.LinearLayoutManager
import com.firstcall.app.R
import com.firstcall.app.data.models.Result
import com.firstcall.app.databinding.ActivitySettingsBinding
import com.firstcall.app.ui.auth.LoginActivity
import com.google.android.material.snackbar.Snackbar

/**
 * Экран настроек приложения
 */
class SettingsActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivitySettingsBinding
    private val viewModel: SettingsViewModel by viewModels { SettingsViewModelFactory(applicationContext) }
    
    private lateinit var sourceAdapter: SourceAdapter
    private lateinit var categoryAdapter: CategoryAdapter
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivitySettingsBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        setupAdapters()
        setupUI()
        observeViewModel()
        
        // Загрузка настроек при создании активности
        viewModel.loadSettings()
    }
    
    private fun setupAdapters() {
        // Настройка адаптера источников
        sourceAdapter = SourceAdapter { source, enabled ->
            viewModel.updateSourceState(source.id, enabled)
        }
        binding.sourcesRecyclerView.apply {
            layoutManager = LinearLayoutManager(this@SettingsActivity)
            adapter = sourceAdapter
        }
        
        // Настройка адаптера категорий
        categoryAdapter = CategoryAdapter { category, enabled ->
            viewModel.updateCategoryState(category.id, enabled)
        }
        binding.categoriesRecyclerView.apply {
            layoutManager = LinearLayoutManager(this@SettingsActivity)
            adapter = categoryAdapter
        }
    }
    
    private fun setupUI() {
        // Настройка слушателей переключателей
        binding.loggingSwitch.setOnCheckedChangeListener { _, isChecked ->
            viewModel.updateLoggingState(isChecked)
        }
        
        binding.autocallSwitch.setOnCheckedChangeListener { _, isChecked ->
            viewModel.updateAutocallState(isChecked)
        }
        
        binding.telegramSwitch.setOnCheckedChangeListener { _, isChecked ->
            viewModel.updateTelegramState(isChecked)
        }
        
        // Настройка кнопки сохранения
        binding.saveButton.setOnClickListener {
            viewModel.saveSettings()
        }
        
        // Настройка кнопки выхода
        binding.logoutButton.setOnClickListener {
            viewModel.logout()
            
            // Переход на экран авторизации
            val intent = Intent(this, LoginActivity::class.java)
            intent.flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
            startActivity(intent)
            finish()
        }
    }
    
    private fun observeViewModel() {
        // Наблюдение за результатами загрузки настроек
        viewModel.settingsResult.observe(this) { result ->
            when (result) {
                is Result.Loading -> {
                    binding.progressBar.isVisible = true
                }
                is Result.Success -> {
                    binding.progressBar.isVisible = false
                    Snackbar.make(
                        binding.root,
                        R.string.connection_established,
                        Snackbar.LENGTH_SHORT
                    ).show()
                }
                is Result.Error -> {
                    binding.progressBar.isVisible = false
                    
                    if (result.code == 403) {
                        // Подписка истекла, переход к авторизации
                        val intent = Intent(this, LoginActivity::class.java)
                        intent.flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
                        startActivity(intent)
                        finish()
                    } else {
                        Snackbar.make(binding.root, result.message, Snackbar.LENGTH_LONG).show()
                    }
                }
            }
        }
        
        // Наблюдение за результатами обновления настроек
        viewModel.updateResult.observe(this) { result ->
            when (result) {
                is Result.Loading -> {
                    binding.progressBar.isVisible = true
                    binding.saveButton.isEnabled = false
                }
                is Result.Success -> {
                    binding.progressBar.isVisible = false
                    binding.saveButton.isEnabled = true
                    
                    Snackbar.make(
                        binding.root,
                        R.string.settings_saved,
                        Snackbar.LENGTH_SHORT
                    ).show()
                }
                is Result.Error -> {
                    binding.progressBar.isVisible = false
                    binding.saveButton.isEnabled = true
                    
                    if (result.code == 403) {
                        // Подписка истекла, переход к авторизации
                        val intent = Intent(this, LoginActivity::class.java)
                        intent.flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
                        startActivity(intent)
                        finish()
                    } else {
                        Snackbar.make(binding.root, result.message, Snackbar.LENGTH_LONG).show()
                    }
                }
            }
        }
        
        // Наблюдение за изменениями источников
        viewModel.sources.observe(this) { sources ->
            sourceAdapter.submitList(sources)
        }
        
        // Наблюдение за изменениями категорий
        viewModel.categories.observe(this) { categories ->
            categoryAdapter.submitList(categories)
        }
        
        // Наблюдение за состоянием логирования
        viewModel.loggingEnabled.observe(this) { enabled ->
            binding.loggingSwitch.isChecked = enabled
        }
        
        // Наблюдение за состоянием автозвонка
        viewModel.autocallEnabled.observe(this) { enabled ->
            binding.autocallSwitch.isChecked = enabled
        }
        
        // Наблюдение за состоянием Telegram уведомлений
        viewModel.telegramEnabled.observe(this) { enabled ->
            binding.telegramSwitch.isChecked = enabled
        }
    }
}
```

### Этап 4: Разработка функционала WebSocket (3-4 дня)

#### Задачи:
- Создание WebSocket клиента
- Реализация фонового сервиса
- Интеграция с системой телефонии

#### Детализация:

1. **WebSocket клиент**

Создайте файл `data/api/WebSocketClient.kt`:
```kotlin
package com.firstcall.app.data.api

import android.os.Handler
import android.os.Looper
import android.util.Log
import com.firstcall.app.data.local.TokenStorage
import kotlinx.coroutines.flow.MutableSharedFlow
import kotlinx.coroutines.flow.SharedFlow
import okhttp3.OkHttpClient
import okhttp3.Request
import okhttp3.Response
import okhttp3.WebSocket
import okhttp3.WebSocketListener
import org.json.JSONObject
import java.util.concurrent.TimeUnit

/**
 * Состояния соединения WebSocket
 */
sealed class ConnectionState {
    object Connected : ConnectionState()
    object Disconnected : ConnectionState()
    data class Error(val message: String) : ConnectionState()
}

/**
 * Команды, получаемые от сервера
 */
sealed class WebSocketCommand {
    data class Call(val phoneNumber: String) : WebSocketCommand()
    object UpdateSettings : WebSocketCommand()
}

/**
 * Клиент для работы с WebSocket
 */
class WebSocketClient(
    private val serverUrl: String,
    private val tokenStorage: TokenStorage
) {
    
    private val TAG = "WebSocketClient"
    
    private var webSocket: WebSocket? = null
    private var isConnecting = false
    private var reconnectAttempts = 0
    private val maxReconnectAttempts = 5
    private val reconnectDelayMillis = 5000L // 5 секунд
    
    private val client = OkHttpClient.Builder()
        .readTimeout(0, TimeUnit.MILLISECONDS) // Без таймаута для WebSocket
        .pingInterval(30, TimeUnit.SECONDS) // Пинг для поддержания соединения
        .build()
    
    private val mainHandler = Handler(Looper.getMainLooper())
    
    // Flow для передачи состояния соединения
    private val _connectionState = MutableSharedFlow<ConnectionState>(replay = 1)
    val connectionState: SharedFlow<ConnectionState> = _connectionState
    
    // Flow для передачи команд от сервера
    private val _commands = MutableSharedFlow<WebSocketCommand>(replay = 0)
    val commands: SharedFlow<WebSocketCommand> = _commands
    
    /**
     * Подключение к WebSocket серверу
     */
    fun connect() {
        if (isConnecting || webSocket != null) {
            return
        }
        
        isConnecting = true
        
        val token = tokenStorage.getAccessToken()
        if (token == null) {
            Log.e(TAG, "Не удалось подключиться: отсутствует токен")
            return
        }
        
        val request = Request.Builder()
            .url("$serverUrl/ws")
            .addHeader("Authorization", "Bearer $token")
            .build()
        
        webSocket = client.newWebSocket(request, object : WebSocketListener() {
            override fun onOpen(webSocket: WebSocket, response: Response) {
                Log.d(TAG, "WebSocket соединение установлено")
                isConnecting = false
                reconnectAttempts = 0
                _connectionState.tryEmit(ConnectionState.Connected)
            }
            
            override fun onMessage(webSocket: WebSocket, text: String) {
                Log.d(TAG, "Получено сообщение: $text")
                
                try {
                    val json = JSONObject(text)
                    when (json.getString("type")) {
                        "call" -> {
                            val phoneNumber = json.getString("phoneNumber")
                            _commands.tryEmit(WebSocketCommand.Call(phoneNumber))
                        }
                        "settings_update" -> {
                            _commands.tryEmit(WebSocketCommand.UpdateSettings)
                        }
                        else -> {
                            Log.w(TAG, "Неизвестный тип команды: ${json.getString("type")}")
                        }
                    }
                } catch (e: Exception) {
                    Log.e(TAG, "Ошибка при обработке сообщения: ${e.message}")
                }
            }
            
            override fun onClosing(webSocket: WebSocket, code: Int, reason: String) {
                Log.d(TAG, "WebSocket закрывается: $code, $reason")
                webSocket.close(1000, null)
                this@WebSocketClient.webSocket = null
                _connectionState.tryEmit(ConnectionState.Disconnected)
            }
            
            override fun onFailure(webSocket: WebSocket, t: Throwable, response: Response?) {
                Log.e(TAG, "Ошибка WebSocket: ${t.message}")
                isConnecting = false
                this@WebSocketClient.webSocket = null
                _connectionState.tryEmit(ConnectionState.Error(t.message ?: "Неизвестная ошибка"))
                
                // Пытаемся переподключиться
                if (reconnectAttempts < maxReconnectAttempts) {
                    reconnectAttempts++
                    mainHandler.postDelayed({ connect() }, reconnectDelayMillis)
                }
            }
        })
    }
    
    /**
     * Отключение от WebSocket сервера
     */
    fun disconnect() {
        webSocket?.close(1000, "Закрыто клиентом")
        webSocket = null
        isConnecting = false
    }
    
    /**
     * Отправка статуса телефона на сервер
     * @param isInCall true, если телефон занят (идет звонок)
     */
    fun sendPhoneStatus(isInCall: Boolean) {
        val ws = webSocket ?: return
        
        try {
            val message = JSONObject().apply {
                put("type", "phone_status")
                put("inCall", isInCall)
            }.toString()
            
            ws.send(message)
            Log.d(TAG, "Отправлен статус телефона: $isInCall")
        } catch (e: Exception) {
            Log.e(TAG, "Ошибка при отправке статуса телефона: ${e.message}")
        }
    }
}
```

2. **Менеджер телефонных звонков**

Создайте файл `ui/service/PhoneCallManager.kt`:
```kotlin
package com.firstcall.app.ui.service

import android.Manifest
import android.content.Context
import android.content.Intent
import android.content.pm.PackageManager
import android.net.Uri
import android.telephony.PhoneStateListener
import android.telephony.TelephonyManager
import android.util.Log
import androidx.core.content.ContextCompat
import com.firstcall.app.data.api.WebSocketClient

/**
 * Менеджер для работы с телефонными звонками
 */
class PhoneCallManager(
    private val context: Context,
    private val webSocketClient: WebSocketClient? = null
) {
    
    private val TAG = "PhoneCallManager"
    
    private val telephonyManager = context.getSystemService(Context.TELEPHONY_SERVICE) as TelephonyManager
    
    // Слушатель состояния телефона
    private val phoneStateListener = object : PhoneStateListener() {
        override fun onCallStateChanged(state: Int, phoneNumber: String?) {
            Log.d(TAG, "Состояние звонка изменено: $state")
            
            val isInCall = state == TelephonyManager.CALL_STATE_OFFHOOK || 
                          state == TelephonyManager.CALL_STATE_RINGING
            
            // Отправка статуса на сервер
            webSocketClient?.sendPhoneStatus(isInCall)
        }
    }
    
    init {
        // Регистрация слушателя состояния телефона
        telephonyManager.listen(phoneStateListener, PhoneStateListener.LISTEN_CALL_STATE)
    }
    
    /**
     * Инициирование звонка на указанный номер
     * @param phoneNumber Номер телефона для звонка
     * @return true, если звонок был инициирован
     */
    fun initiateCall(phoneNumber: String): Boolean {
        Log.d(TAG, "Попытка позвонить на номер: $phoneNumber")
        
        // Проверка разрешения на совершение звонков
        if (ContextCompat.checkSelfPermission(context, Manifest.permission.CALL_PHONE) == 
            PackageManager.PERMISSION_GRANTED) {
            
            try {
                // Создание Intent для звонка
                val intent = Intent(Intent.ACTION_CALL).apply {
                    data = Uri.parse("tel:$phoneNumber")
                    flags = Intent.FLAG_ACTIVITY_NEW_TASK
                }
                
                // Запуск звонка
                context.startActivity(intent)
                return true
            } catch (e: Exception) {
                Log.e(TAG, "Ошибка при инициировании звонка: ${e.message}")
                return false
            }
        } else {
            Log.e(TAG, "Нет разрешения на совершение звонков")
            return false
        }
    }
    
    /**
     * Освобождение ресурсов
     */
    fun cleanup() {
        // Отписка от слушателя состояния телефона
        telephonyManager.listen(phoneStateListener, PhoneStateListener.LISTEN_NONE)
    }
}
```

3. **Фоновый сервис для звонков**

Создайте файл `ui/service/CallService.kt`:
```kotlin
package com.firstcall.app.ui.service

import android.app.Notification
import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.Service
import android.content.Intent
import android.content.pm.ServiceInfo
import android.os.Build
import android.os.IBinder
import android.util.Log
import androidx.core.app.NotificationCompat
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import com.firstcall.app.R
import com.firstcall.app.data.api.ConnectionState
import com.firstcall.app.data.api.WebSocketClient
import com.firstcall.app.data.api.WebSocketCommand
import com.firstcall.app.data.local.TokenStorage
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.SupervisorJob
import kotlinx.coroutines.cancel
import kotlinx.coroutines.flow.launchIn
import kotlinx.coroutines.flow.onEach
import kotlinx.coroutines.launch

/**
 * Фоновый сервис для обработки звонков через WebSocket
 */
class CallService : Service() {
    
    private val TAG = "CallService"
    
    // Константы для
