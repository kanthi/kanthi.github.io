+++
title = "Rails Hotwire Native: Building Hybrid Mobile Apps with Rails and Turbo Native"
date = "2025-07-28T17:00:00+05:30"
author = ""
draft = true
authorTwitter = "" #do not include @
cover = ""
tags = ["rails", "hotwire", "turbo-native", "mobile", "hybrid-apps", "ios", "android", "ruby-on-rails"]
keywords = ["rails hotwire native", "turbo native", "rails mobile app", "hybrid mobile development", "hotwire turbo", "rails ios android", "mobile web apps"]
description = "Complete guide to building hybrid mobile applications using Rails Hotwire Native. Learn how to create native mobile apps powered by your Rails web application with Turbo Native."
showFullContent = false
readingTime = false
hideComments = false
+++

Rails Hotwire Native represents a revolutionary approach to mobile app development, allowing you to build native mobile applications using your existing Rails web application as the foundation. By leveraging Turbo Native, you can create hybrid apps that feel native while maintaining a single codebase and leveraging the full power of Rails.

## What is Rails Hotwire Native?

### Hotwire Overview
Hotwire (HTML Over The Wire) is a collection of techniques for building modern web applications without much JavaScript:
- **Turbo**: Accelerates links and form submissions, divides complex pages into components
- **Stimulus**: A modest JavaScript framework for the HTML you already have
- **Strada**: Standardizes the way that web and native parts of a mobile hybrid application talk to each other

### Turbo Native
Turbo Native is a framework that lets you wrap your Rails web application in a native shell, providing:
- **Native Navigation**: Native navigation controllers and tab bars
- **Native Performance**: Fast transitions and native UI elements
- **Web Content**: Your existing Rails views rendered in native web views
- **Bridge Communication**: Two-way communication between native and web layers

## Why Choose Hotwire Native?

### 1. **Leverage Existing Rails App**
- Use your current Rails application as the mobile backend
- No need to build separate API endpoints
- Maintain feature parity between web and mobile

### 2. **Rapid Development**
- Build mobile apps without learning Swift/Kotlin
- Reuse existing Rails views and logic
- Single team can maintain web and mobile

### 3. **Native Feel**
- Native navigation and transitions
- Platform-specific UI elements
- Access to device capabilities

### 4. **Cost Effective**
- Minimal additional development resources
- Shared codebase reduces maintenance
- Faster time-to-market

## Rails Application Setup

### Prerequisites

```bash
# Ensure you have Rails 7+ with Hotwire
rails --version  # Should be 7.0+

# Install required tools
gem install rails
brew install node  # For Stimulus and asset compilation
```

### Create New Rails Application

```bash
# Create new Rails app with Hotwire (default in Rails 7)
rails new HotwireNativeApp --database=postgresql

cd HotwireNativeApp

# Or add to existing Rails app
bundle add turbo-rails stimulus-rails
```

### Configure Hotwire in Rails

```ruby
# Gemfile
gem 'turbo-rails'
gem 'stimulus-rails'
gem 'redis', '~> 4.0'  # For ActionCable
gem 'image_processing', '~> 1.2'  # For Active Storage variants

group :development, :test do
  gem 'debug', platforms: %i[ mri mingw x64_mingw ]
end
```

```bash
# Install dependencies
bundle install

# Generate Turbo and Stimulus configuration
rails turbo:install
rails stimulus:install

# Setup database
rails db:create
rails db:migrate
```

### Application Configuration

```ruby
# config/application.rb
require_relative "boot"
require "rails/all"

Bundler.require(*Rails.groups)

module HotwireNativeApp
  class Application < Rails::Application
    config.load_defaults 7.1

    # Enable Hotwire Native features
    config.force_ssl = true if Rails.env.production?

    # Configure session store for mobile apps
    config.session_store :cookie_store,
      key: '_hotwire_native_session',
      secure: Rails.env.production?,
      same_site: :lax
  end
end
```

```ruby
# config/routes.rb
Rails.application.routes.draw do
  root 'home#index'

  resources :posts do
    member do
      patch :toggle_favorite
    end
  end

  resources :users, only: [:show, :edit, :update]
  resources :sessions, only: [:new, :create, :destroy]

  # Native-specific routes
  namespace :native do
    resources :posts, only: [:index, :show]
    get 'profile', to: 'users#profile'
  end
end
```

## Building the Rails Backend

### Models and Controllers

```ruby
# app/models/post.rb
class Post < ApplicationRecord
  belongs_to :user
  has_many :favorites, dependent: :destroy
  has_many_attached :images

  validates :title, presence: true
  validates :content, presence: true

  scope :recent, -> { order(created_at: :desc) }
  scope :published, -> { where(published: true) }

  def favorited_by?(user)
    return false unless user
    favorites.exists?(user: user)
  end

  broadcasts_to ->(post) { "posts" }, inserts_by: :prepend
end
```

```ruby
# app/models/user.rb
class User < ApplicationRecord
  has_secure_password

  has_many :posts, dependent: :destroy
  has_many :favorites, dependent: :destroy
  has_one_attached :avatar

  validates :email, presence: true, uniqueness: true
  validates :name, presence: true
end
```

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  before_action :authenticate_user!
  before_action :set_current_user

  private

  def authenticate_user!
    redirect_to new_session_path unless current_user
  end

  def current_user
    @current_user ||= User.find(session[:user_id]) if session[:user_id]
  end

  def set_current_user
    Current.user = current_user
  end

  # Detect if request is from native app
  def native_app?
    request.user_agent&.include?('Turbo Native')
  end

  helper_method :current_user, :native_app?
end
```

```ruby
# app/controllers/posts_controller.rb
class PostsController < ApplicationController
  before_action :set_post, only: [:show, :edit, :update, :destroy, :toggle_favorite]

  def index
    @posts = Post.published.recent.includes(:user, images_attachments: :blob)

    respond_to do |format|
      format.html
      format.turbo_stream
    end
  end

  def show
    respond_to do |format|
      format.html
      format.turbo_stream
    end
  end

  def new
    @post = current_user.posts.build
  end

  def create
    @post = current_user.posts.build(post_params)

    if @post.save
      respond_to do |format|
        format.html { redirect_to @post, notice: 'Post created successfully!' }
        format.turbo_stream { flash.now[:notice] = 'Post created successfully!' }
      end
    else
      render :new, status: :unprocessable_entity
    end
  end

  def edit
  end

  def update
    if @post.update(post_params)
      respond_to do |format|
        format.html { redirect_to @post, notice: 'Post updated successfully!' }
        format.turbo_stream { flash.now[:notice] = 'Post updated successfully!' }
      end
    else
      render :edit, status: :unprocessable_entity
    end
  end

  def destroy
    @post.destroy

    respond_to do |format|
      format.html { redirect_to posts_path, notice: 'Post deleted successfully!' }
      format.turbo_stream { flash.now[:notice] = 'Post deleted successfully!' }
    end
  end

  def toggle_favorite
    favorite = current_user.favorites.find_by(post: @post)

    if favorite
      favorite.destroy
      favorited = false
    else
      current_user.favorites.create(post: @post)
      favorited = true
    end

    respond_to do |format|
      format.turbo_stream do
        render turbo_stream: turbo_stream.replace(
          "favorite_#{@post.id}",
          partial: 'posts/favorite_button',
          locals: { post: @post, favorited: favorited }
        )
      end
    end
  end

  private

  def set_post
    @post = Post.find(params[:id])
  end

  def post_params
    params.require(:post).permit(:title, :content, :published, images: [])
  end
end
```

### Views with Turbo Frames

```erb
<!-- app/views/layouts/application.html.erb -->
<!DOCTYPE html>
<html>
  <head>
    <title>Hotwire Native App</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>

    <!-- Native app detection -->
    <% if native_app? %>
      <meta name="turbo-native-bridge" content="true">
      <style>
        /* Native-specific styles */
        .web-only { display: none !important; }
        .native-navigation { display: block; }
      </style>
    <% else %>
      <style>
        .native-only { display: none !important; }
      </style>
    <% end %>
  </head>

  <body>
    <% unless native_app? %>
      <nav class="navbar">
        <%= link_to "Home", root_path, class: "nav-link" %>
        <%= link_to "Posts", posts_path, class: "nav-link" %>
        <% if current_user %>
          <%= link_to "Profile", user_path(current_user), class: "nav-link" %>
          <%= link_to "Logout", session_path(current_user), method: :delete, class: "nav-link" %>
        <% else %>
          <%= link_to "Login", new_session_path, class: "nav-link" %>
        <% end %>
      </nav>
    <% end %>

    <main class="container">
      <%= render 'shared/flash_messages' %>
      <%= yield %>
    </main>

    <!-- Turbo Stream connection for real-time updates -->
    <%= turbo_stream_from "posts" if current_user %>
  </body>
</html>
```

```erb
<!-- app/views/posts/index.html.erb -->
<div class="posts-header">
  <h1>Recent Posts</h1>
  <%= link_to "New Post", new_post_path,
      class: "btn btn-primary",
      data: { turbo_frame: "modal" } %>
</div>

<%= turbo_frame_tag "posts" do %>
  <div class="posts-grid">
    <%= render @posts %>
  </div>
<% end %>

<!-- Modal frame for new/edit forms -->
<%= turbo_frame_tag "modal" %>
```

```erb
<!-- app/views/posts/_post.html.erb -->
<%= turbo_frame_tag dom_id(post), class: "post-card" do %>
  <div class="post-header">
    <div class="post-author">
      <% if post.user.avatar.attached? %>
        <%= image_tag post.user.avatar, class: "avatar" %>
      <% end %>
      <span><%= post.user.name %></span>
    </div>

    <div class="post-actions">
      <%= render 'posts/favorite_button', post: post, favorited: post.favorited_by?(current_user) %>

      <% if post.user == current_user %>
        <%= link_to "Edit", edit_post_path(post),
            class: "btn btn-sm btn-outline",
            data: { turbo_frame: "modal" } %>
        <%= link_to "Delete", post_path(post),
            method: :delete,
            class: "btn btn-sm btn-danger",
            data: {
              turbo_method: :delete,
              turbo_confirm: "Are you sure?",
              turbo_frame: "_top"
            } %>
      <% end %>
    </div>
  </div>

  <div class="post-content">
    <h3><%= link_to post.title, post_path(post) %></h3>
    <p><%= truncate(post.content, length: 150) %></p>

    <% if post.images.attached? %>
      <div class="post-images">
        <% post.images.each do |image| %>
          <%= image_tag image, class: "post-image" %>
        <% end %>
      </div>
    <% end %>
  </div>

  <div class="post-footer">
    <small class="text-muted">
      <%= time_ago_in_words(post.created_at) %> ago
    </small>
  </div>
<% end %>
```

```erb
<!-- app/views/posts/_favorite_button.html.erb -->
<%= turbo_frame_tag "favorite_#{post.id}" do %>
  <%= link_to toggle_favorite_post_path(post),
      method: :patch,
      class: "favorite-btn #{'favorited' if favorited}",
      data: { turbo_frame: "favorite_#{post.id}" } do %>
    <% if favorited %>
      ❤️ Favorited
    <% else %>
      🤍 Favorite
    <% end %>
  <% end %>
<% end %>
```

### Stimulus Controllers

```javascript
// app/javascript/controllers/posts_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["form", "content"]

  connect() {
    console.log("Posts controller connected")
  }

  refresh() {
    // Refresh posts list
    this.element.reload()
  }

  toggleFavorite(event) {
    event.preventDefault()

    const button = event.currentTarget
    const originalText = button.textContent

    button.textContent = "Loading..."
    button.disabled = true

    // The form submission will handle the actual toggle
    // This is just for immediate UI feedback
    setTimeout(() => {
      button.disabled = false
    }, 500)
  }
}
```

```javascript
// app/javascript/controllers/modal_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["dialog"]

  connect() {
    this.element.addEventListener("turbo:frame-load", this.frameLoaded.bind(this))
  }

  frameLoaded() {
    if (this.hasDialogTarget && this.dialogTarget.innerHTML.trim()) {
      this.dialogTarget.showModal()
    }
  }

  close() {
    if (this.hasDialogTarget) {
      this.dialogTarget.close()
      this.dialogTarget.innerHTML = ""
    }
  }

  backdropClick(event) {
    if (event.target === this.dialogTarget) {
      this.close()
    }
  }
}
```

## iOS Native App Setup

### Prerequisites

```bash
# Install Xcode from Mac App Store
# Install CocoaPods
sudo gem install cocoapods

# Install Turbo Native iOS
# This will be done through CocoaPods in the iOS project
```

### Create iOS Project

```bash
# Create new iOS project directory
mkdir HotwireNativeApp-iOS
cd HotwireNativeApp-iOS

# Initialize Xcode project (or create through Xcode)
# File > New > Project > iOS > App
# Choose Swift as language
```

### Podfile Configuration

```ruby
# Podfile
platform :ios, '14.0'
use_frameworks!

target 'HotwireNativeApp' do
  pod 'Turbo', '~> 7.0'
  pod 'Strada', '~> 1.0'

  target 'HotwireNativeAppTests' do
    inherit! :search_paths
  end
end

post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '14.0'
    end
  end
end
```

```bash
# Install pods
pod install

# Open workspace (not .xcodeproj)
open HotwireNativeApp.xcworkspace
```

### Main App Delegate

```swift
// AppDelegate.swift
import UIKit
import Turbo

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

        window = UIWindow(frame: UIScreen.main.bounds)
        window?.rootViewController = MainTabBarController()
        window?.makeKeyAndVisible()

        return true
    }
}
```

### Main Tab Bar Controller

```swift
// MainTabBarController.swift
import UIKit
import Turbo

class MainTabBarController: UITabBarController {
    private let homeNavigationController = TurboNavigationController()
    private let postsNavigationController = TurboNavigationController()
    private let profileNavigationController = TurboNavigationController()

    override func viewDidLoad() {
        super.viewDidLoad()

        setupTabBar()
        setupNavigationControllers()
    }

    private func setupTabBar() {
        tabBar.tintColor = .systemBlue
        tabBar.backgroundColor = .systemBackground
    }

    private func setupNavigationControllers() {
        // Home Tab
        homeNavigationController.tabBarItem = UITabBarItem(
            title: "Home",
            image: UIImage(systemName: "house"),
            selectedImage: UIImage(systemName: "house.fill")
        )

        // Posts Tab
        postsNavigationController.tabBarItem = UITabBarItem(
            title: "Posts",
            image: UIImage(systemName: "doc.text"),
            selectedImage: UIImage(systemName: "doc.text.fill")
        )

        // Profile Tab
        profileNavigationController.tabBarItem = UITabBarItem(
            title: "Profile",
            image: UIImage(systemName: "person"),
            selectedImage: UIImage(systemName: "person.fill")
        )

        viewControllers = [
            homeNavigationController,
            postsNavigationController,
            profileNavigationController
        ]

        // Start with home tab
        selectedIndex = 0

        // Load initial URLs
        loadInitialURLs()
    }

    private func loadInitialURLs() {
        let baseURL = URL(string: "https://your-app.com")!

        homeNavigationController.route(url: baseURL.appendingPathComponent("/"))
        postsNavigationController.route(url: baseURL.appendingPathComponent("/posts"))
        profileNavigationController.route(url: baseURL.appendingPathComponent("/native/profile"))
    }
}
```

### Turbo Navigation Controller

```swift
// TurboNavigationController.swift
import UIKit
import Turbo

class TurboNavigationController: UINavigationController {
    private lazy var session = Session()

    override func viewDidLoad() {
        super.viewDidLoad()

        session.delegate = self
        session.pathConfiguration = PathConfiguration(sources: [
            .server(url: URL(string: "https://your-app.com/turbo_native/path_configuration.json")!)
        ])
    }

    func route(url: URL) {
        let proposal = VisitProposal(url: url)
        session.visit(proposal)
    }
}

// MARK: - SessionDelegate
extension TurboNavigationController: SessionDelegate {
    func session(_ session: Session, didProposeVisit proposal: VisitProposal) {
        session.visit(proposal)
    }

    func session(_ session: Session, didFailRequestForVisitable visitable: Visitable, error: Error) {
        print("Failed to load: \(error)")

        if let viewController = visitable as? UIViewController {
            let alert = UIAlertController(
                title: "Error",
                message: "Failed to load page. Please try again.",
                preferredStyle: .alert
            )
            alert.addAction(UIAlertAction(title: "OK", style: .default))
            viewController.present(alert, animated: true)
        }
    }

    func sessionDidStartRequest(_ session: Session) {
        // Show loading indicator
    }

    func sessionDidFinishRequest(_ session: Session) {
        // Hide loading indicator
    }

    func session(_ session: Session, openExternalURL url: URL) {
        UIApplication.shared.open(url)
    }

    func sessionDidLoadWebView(_ session: Session) {
        session.webView.navigationDelegate = self
    }
}

// MARK: - WKNavigationDelegate
extension TurboNavigationController: WKNavigationDelegate {
    func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {

        // Handle native-specific URLs
        if let url = navigationAction.request.url,
           url.absoluteString.contains("/native/") {
            handleNativeRoute(url: url)
            decisionHandler(.cancel)
            return
        }

        decisionHandler(.allow)
    }

    private func handleNativeRoute(url: URL) {
        let path = url.path

        switch path {
        case "/native/camera":
            presentCameraController()
        case "/native/location":
            requestLocationPermission()
        default:
            break
        }
    }

    private func presentCameraController() {
        let cameraController = CameraViewController()
        present(cameraController, animated: true)
    }

    private func requestLocationPermission() {
        // Handle location permission request
    }
}
```

### Custom Native Controllers

```swift
// CameraViewController.swift
import UIKit
import AVFoundation

class CameraViewController: UIViewController {
    private var captureSession: AVCaptureSession!
    private var previewLayer: AVCaptureVideoPreviewLayer!

    override func viewDidLoad() {
        super.viewDidLoad()

        setupCamera()
        setupUI()
    }

    private func setupCamera() {
        captureSession = AVCaptureSession()

        guard let videoCaptureDevice = AVCaptureDevice.default(for: .video) else { return }
        let videoInput: AVCaptureDeviceInput

        do {
            videoInput = try AVCaptureDeviceInput(device: videoCaptureDevice)
        } catch {
            return
        }

        if captureSession.canAddInput(videoInput) {
            captureSession.addInput(videoInput)
        } else {
            return
        }

        previewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
        previewLayer.frame = view.layer.bounds
        previewLayer.videoGravity = .resizeAspectFill
        view.layer.addSublayer(previewLayer)

        captureSession.startRunning()
    }

    private func setupUI() {
        // Add camera controls
        let captureButton = UIButton(type: .system)
        captureButton.setTitle("Capture", for: .normal)
        captureButton.backgroundColor = .systemBlue
        captureButton.layer.cornerRadius = 25
        captureButton.addTarget(self, action: #selector(capturePhoto), for: .touchUpInside)

        view.addSubview(captureButton)
        captureButton.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            captureButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            captureButton.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -20),
            captureButton.widthAnchor.constraint(equalToConstant: 100),
            captureButton.heightAnchor.constraint(equalToConstant: 50)
        ])

        // Close button
        let closeButton = UIButton(type: .system)
        closeButton.setTitle("Close", for: .normal)
        closeButton.addTarget(self, action: #selector(closeCamera), for: .touchUpInside)

        view.addSubview(closeButton)
        closeButton.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            closeButton.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            closeButton.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20)
        ])
    }

    @objc private func capturePhoto() {
        // Implement photo capture
        print("Photo captured")
    }

    @objc private func closeCamera() {
        dismiss(animated: true)
    }
}
```

### Path Configuration

```json
// public/turbo_native/path_configuration.json
{
  "settings": {
    "screenshots_enabled": true
  },
  "rules": [
    {
      "patterns": ["/"],
      "properties": {
        "context": "default",
        "uri": "turbo://fragment/web",
        "pull_to_refresh_enabled": true
      }
    },
    {
      "patterns": ["/posts"],
      "properties": {
        "context": "default",
        "uri": "turbo://fragment/web",
        "pull_to_refresh_enabled": true
      }
    },
    {
      "patterns": ["/posts/new", "/posts/*/edit"],
      "properties": {
        "context": "modal",
        "uri": "turbo://fragment/web",
        "presentation": "modal"
      }
    },
    {
      "patterns": ["/native/camera"],
      "properties": {
        "uri": "turbo://fragment/native/camera"
      }
    }
  ]
}
```

## Android Native App Setup

### Prerequisites

```bash
# Install Android Studio
# Install Java 11 or higher
java --version

# Create new Android project
# File > New > New Project > Empty Activity
# Choose Kotlin as language
# Minimum SDK: API 24 (Android 7.0)
```

### Gradle Configuration

```kotlin
// app/build.gradle
android {
    compileSdk 34

    defaultConfig {
        applicationId "com.yourcompany.hotwirenativeapp"
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0"
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_11
        targetCompatibility JavaVersion.VERSION_11
    }

    kotlinOptions {
        jvmTarget = "11"
    }
}

dependencies {
    implementation 'dev.hotwire:turbo-android:7.0.0'
    implementation 'dev.hotwire:strada-android:1.0.0'

    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.11.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    implementation 'androidx.navigation:navigation-fragment-ktx:2.7.6'
    implementation 'androidx.navigation:navigation-ui-ktx:2.7.6'
}
```

### Main Activity

```kotlin
// MainActivity.kt
package com.yourcompany.hotwirenativeapp

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.navigation.findNavController
import androidx.navigation.ui.setupWithNavController
import com.google.android.material.bottomnavigation.BottomNavigationView
import dev.hotwire.turbo.config.TurboPathConfiguration
import dev.hotwire.turbo.session.TurboSessionNavHostFragment

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        setupBottomNavigation()
        setupTurboPathConfiguration()
    }

    private fun setupBottomNavigation() {
        val navView: BottomNavigationView = findViewById(R.id.nav_view)
        val navController = findNavController(R.id.nav_host_fragment)
        navView.setupWithNavController(navController)
    }

    private fun setupTurboPathConfiguration() {
        TurboPathConfiguration.load(
            context = this,
            location = TurboPathConfiguration.Location.REMOTE,
            url = "https://your-app.com/turbo_native/path_configuration.json"
        )
    }
}
```

### Navigation Host Fragment

```kotlin
// MainNavHostFragment.kt
package com.yourcompany.hotwirenativeapp

import androidx.fragment.app.Fragment
import dev.hotwire.turbo.session.TurboSessionNavHostFragment
import kotlin.reflect.KClass

class MainNavHostFragment : TurboSessionNavHostFragment() {
    override val sessionName = "main"

    override val startLocation = "https://your-app.com"

    override val registeredActivities: List<KClass<out HotwireActivity>>
        get() = listOf()

    override val registeredFragments: List<KClass<out Fragment>>
        get() = listOf(
            WebFragment::class,
            CameraFragment::class
        )

    override fun onFormSubmissionStarted(location: String) {
        // Show loading indicator
    }

    override fun onFormSubmissionFinished(location: String) {
        // Hide loading indicator
    }
}
```

### Web Fragment

```kotlin
// WebFragment.kt
package com.yourcompany.hotwirenativeapp

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import dev.hotwire.turbo.fragments.TurboWebFragment
import dev.hotwire.turbo.views.TurboWebView

class WebFragment : TurboWebFragment() {
    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_web, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        setupWebView()
    }

    private fun setupWebView() {
        webView.settings.apply {
            javaScriptEnabled = true
            domStorageEnabled = true
        }

        // Add JavaScript interface for native communication
        webView.addJavascriptInterface(
            NativeBridge(this),
            "NativeBridge"
        )
    }

    override fun onVisitCompleted(completedSuccessfully: Boolean) {
        if (completedSuccessfully) {
            // Inject native bridge JavaScript
            webView.evaluateJavascript("""
                window.nativeBridge = {
                    openCamera: function() {
                        NativeBridge.openCamera();
                    },
                    getLocation: function() {
                        NativeBridge.getLocation();
                    }
                };
            """.trimIndent(), null)
        }
    }
}
```

### Native Bridge

```kotlin
// NativeBridge.kt
package com.yourcompany.hotwirenativeapp

import android.webkit.JavascriptInterface
import androidx.navigation.fragment.findNavController

class NativeBridge(private val fragment: WebFragment) {

    @JavascriptInterface
    fun openCamera() {
        fragment.activity?.runOnUiThread {
            // Navigate to camera fragment
            fragment.findNavController().navigate(R.id.action_web_to_camera)
        }
    }

    @JavascriptInterface
    fun getLocation() {
        fragment.activity?.runOnUiThread {
            // Handle location request
            LocationHelper.requestLocation(fragment.requireContext()) { location ->
                // Send location back to web view
                fragment.webView.evaluateJavascript("""
                    window.dispatchEvent(new CustomEvent('native:location', {
                        detail: {
                            latitude: ${location.latitude},
                            longitude: ${location.longitude}
                        }
                    }));
                """.trimIndent(), null)
            }
        }
    }
}
```

### Camera Fragment

```kotlin
// CameraFragment.kt
package com.yourcompany.hotwirenativeapp

import android.Manifest
import android.content.pm.PackageManager
import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.camera.core.*
import androidx.camera.lifecycle.ProcessCameraProvider
import androidx.core.app.ActivityCompat
import androidx.core.content.ContextCompat
import androidx.fragment.app.Fragment
import androidx.navigation.fragment.findNavController
import java.util.concurrent.ExecutorService
import java.util.concurrent.Executors

class CameraFragment : Fragment() {
    private var imageCapture: ImageCapture? = null
    private lateinit var cameraExecutor: ExecutorService

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_camera, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        if (allPermissionsGranted()) {
            startCamera()
        } else {
            ActivityCompat.requestPermissions(
                requireActivity(),
                REQUIRED_PERMISSIONS,
                REQUEST_CODE_PERMISSIONS
            )
        }

        cameraExecutor = Executors.newSingleThreadExecutor()

        setupUI()
    }

    private fun setupUI() {
        view?.findViewById<View>(R.id.camera_capture_button)?.setOnClickListener {
            takePhoto()
        }

        view?.findViewById<View>(R.id.camera_close_button)?.setOnClickListener {
            findNavController().navigateUp()
        }
    }

    private fun startCamera() {
        val cameraProviderFuture = ProcessCameraProvider.getInstance(requireContext())

        cameraProviderFuture.addListener({
            val cameraProvider: ProcessCameraProvider = cameraProviderFuture.get()

            val preview = Preview.Builder().build().also {
                it.setSurfaceProvider(viewFinder.surfaceProvider)
            }

            imageCapture = ImageCapture.Builder().build()

            val cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA

            try {
                cameraProvider.unbindAll()
                cameraProvider.bindToLifecycle(
                    this, cameraSelector, preview, imageCapture
                )
            } catch (exc: Exception) {
                // Handle error
            }
        }, ContextCompat.getMainExecutor(requireContext()))
    }

    private fun takePhoto() {
        // Implement photo capture logic
    }

    private fun allPermissionsGranted() = REQUIRED_PERMISSIONS.all {
        ContextCompat.checkSelfPermission(requireContext(), it) == PackageManager.PERMISSION_GRANTED
    }

    override fun onDestroy() {
        super.onDestroy()
        cameraExecutor.shutdown()
    }

    companion object {
        private const val REQUEST_CODE_PERMISSIONS = 10
        private val REQUIRED_PERMISSIONS = arrayOf(Manifest.permission.CAMERA)
    }
}
```

## Advanced Features

### Real-time Updates with ActionCable

```ruby
# app/channels/posts_channel.rb
class PostsChannel < ApplicationCable::Channel
  def subscribed
    stream_from "posts"
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end
end
```

```javascript
// app/javascript/channels/posts_channel.js
import consumer from "channels/consumer"

consumer.subscriptions.create("PostsChannel", {
  connected() {
    console.log("Connected to PostsChannel")
  },

  disconnected() {
    console.log("Disconnected from PostsChannel")
  },

  received(data) {
    console.log("Received data:", data)
    // Handle real-time updates
  }
})
```

### Push Notifications

```ruby
# app/controllers/notifications_controller.rb
class NotificationsController < ApplicationController
  def create
    # Send push notification to mobile apps
    PushNotificationService.send_to_user(
      current_user,
      title: params[:title],
      body: params[:body],
      data: params[:data]
    )

    head :ok
  end
end
```

### Offline Support

```javascript
// app/javascript/controllers/offline_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  connect() {
    window.addEventListener('online', this.handleOnline.bind(this))
    window.addEventListener('offline', this.handleOffline.bind(this))

    this.checkConnectionStatus()
  }

  handleOnline() {
    this.element.classList.remove('offline')
    this.syncPendingActions()
  }

  handleOffline() {
    this.element.classList.add('offline')
    this.showOfflineMessage()
  }

  syncPendingActions() {
    // Sync any pending actions when back online
    const pendingActions = JSON.parse(localStorage.getItem('pendingActions') || '[]')

    pendingActions.forEach(action => {
      // Replay actions
      this.replayAction(action)
    })

    localStorage.removeItem('pendingActions')
  }

  checkConnectionStatus() {
    if (!navigator.onLine) {
      this.handleOffline()
    }
  }
}
```

## Testing

### Rails Testing

```ruby
# test/system/posts_test.rb
require "application_system_test_case"

class PostsTest < ApplicationSystemTestCase
  setup do
    @user = users(:one)
    login_as(@user)
  end

  test "creating a post" do
    visit posts_path

    click_on "New Post"

    within_frame "modal" do
      fill_in "Title", with: "Test Post"
      fill_in "Content", with: "This is a test post"
      click_on "Create Post"
    end

    assert_text "Test Post"
    assert_text "Post created successfully!"
  end

  test "favoriting a post" do
    post = posts(:one)
    visit posts_path

    within "##{dom_id(post)}" do
      click_on "Favorite"
      assert_text "Favorited"
    end
  end
end
```

### iOS Testing

```swift
// HotwireNativeAppTests/NavigationTests.swift
import XCTest
@testable import HotwireNativeApp

class NavigationTests: XCTestCase {
    var app: XCUIApplication!

    override func setUpWithError() throws {
        continueAfterFailure = false
        app = XCUIApplication()
        app.launch()
    }

    func testTabNavigation() throws {
        // Test tab bar navigation
        let tabBar = app.tabBars.firstMatch
        XCTAssertTrue(tabBar.exists)

        // Test Posts tab
        tabBar.buttons["Posts"].tap()
        XCTAssertTrue(app.staticTexts["Recent Posts"].exists)

        // Test Profile tab
        tabBar.buttons["Profile"].tap()
        XCTAssertTrue(app.navigationBars["Profile"].exists)
    }

    func testWebViewLoading() throws {
        let webView = app.webViews.firstMatch
        XCTAssertTrue(webView.waitForExistence(timeout: 5))
    }
}
```

## Deployment

### Rails Deployment

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Setup Ruby
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: 3.2
        bundler-cache: true

    - name: Setup Node
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: |
        bundle install
        npm install

    - name: Precompile assets
      run: |
        RAILS_ENV=production bundle exec rails assets:precompile

    - name: Deploy with Kamal
      run: |
        bundle exec kamal deploy
      env:
        RAILS_MASTER_KEY: ${{ secrets.RAILS_MASTER_KEY }}
```

### iOS App Store Deployment

```bash
# Build for App Store
# In Xcode:
# 1. Select "Any iOS Device" as target
# 2. Product > Archive
# 3. Distribute App > App Store Connect
# 4. Upload to App Store Connect
# 5. Submit for review through App Store Connect
```

### Android Play Store Deployment

```bash
# Generate signed APK
# In Android Studio:
# 1. Build > Generate Signed Bundle/APK
# 2. Choose Android App Bundle
# 3. Create or select keystore
# 4. Build release bundle
# 5. Upload to Google Play Console
```

## Best Practices

### Performance Optimization

```ruby
# app/controllers/concerns/turbo_native_optimizations.rb
module TurboNativeOptimizations
  extend ActiveSupport::Concern

  included do
    before_action :optimize_for_native, if: :native_app?
  end

  private

  def optimize_for_native
    # Reduce payload size for mobile
    response.headers['Cache-Control'] = 'public, max-age=300'

    # Compress responses
    request.env['HTTP_ACCEPT_ENCODING'] = 'gzip'
  end

  def native_app?
    request.user_agent&.include?('Turbo Native')
  end
end
```

### Security Considerations

```ruby
# config/application.rb
config.force_ssl = true if Rails.env.production?

# Content Security Policy for native apps
config.content_security_policy do |policy|
  policy.default_src :self
  policy.script_src :self, :unsafe_inline if Rails.env.development?
  policy.connect_src :self, 'ws:', 'wss:'
  policy.img_src :self, :data, :blob
end
```

### Error Handling

```swift
// iOS Error Handling
extension TurboNavigationController {
    func handleError(_ error: Error, for visitable: Visitable) {
        let alert = UIAlertController(
            title: "Connection Error",
            message: "Please check your internet connection and try again.",
            preferredStyle: .alert
        )

        alert.addAction(UIAlertAction(title: "Retry", style: .default) { _ in
            if let url = visitable.visitableURL {
                self.route(url: url)
            }
        })

        alert.addAction(UIAlertAction(title: "Cancel", style: .cancel))

        present(alert, animated: true)
    }
}
```

## Conclusion

Rails Hotwire Native represents a paradigm shift in mobile app development, offering:

**Key Benefits:**
- **Unified Codebase**: Single Rails application serves web and mobile
- **Native Performance**: Native navigation and UI elements
- **Rapid Development**: Leverage existing Rails skills and patterns
- **Cost Effective**: Reduced development and maintenance overhead

**What You've Built:**
- Complete Rails application with Hotwire integration
- Native iOS app with Turbo Native
- Native Android app with Turbo Android
- Real-time updates with ActionCable
- Native device integration (camera, location)

**Production Considerations:**
- Performance optimization for mobile networks
- Security best practices for hybrid apps
- App store deployment processes
- Error handling and offline support

Rails Hotwire Native enables teams to build sophisticated mobile applications without the complexity of maintaining separate native codebases. By leveraging the power of Rails and the native capabilities of mobile platforms, you can deliver exceptional user experiences while maintaining development efficiency.

The future of mobile development is hybrid, and Rails Hotwire Native provides the perfect balance of web development productivity and native mobile performance.