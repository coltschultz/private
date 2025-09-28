# HomeCook Connect - App Development Plan

## 🍳 App Overview

HomeCook Connect is a mobile platform connecting home cooks with food lovers, enabling people to share homemade meals with their community. Unlike traditional food delivery apps, this platform focuses on planned, artisanal home cooking with advance notice requirements.

### Core Value Proposition
- **For Cooks**: Monetize culinary skills from home with flexible scheduling
- **For Eaters**: Access authentic, homemade meals with personal touch
- **Community**: Build local food communities with trust and quality

## 👥 User Types & Key Features

### Cooking Users (Sellers)
- **Menu Management**: CRUD operations for dishes with photos, descriptions, ingredients
- **Dynamic Pricing**: Base price + quantity scaling (bulk discounts/premiums)
- **Schedule Control**: Set advance notice requirements (1-48 hours)
- **Order Management**: Accept/decline orders, communicate with buyers
- **Availability Windows**: Define when they cook and deliver/pickup

### Eating Users (Buyers)
- **Discovery**: Browse nearby available meals with filters
- **Ordering**: Select quantities, schedule pickup/delivery
- **Communication**: Message cooks about special requests
- **Reviews**: Rate meals and cooking experiences

## 🛠 Beginner-Friendly Technology Stack

Given your React/Firebase/Express/Node background, here's a stack that leverages your existing skills:

### Frontend - Mobile App
- **Expo with React Native** - Easiest entry into mobile development
  - Similar to React web development you know
  - Expo handles complex native configurations
  - Built-in components for camera, location, notifications
- **JavaScript first, TypeScript later** - Start familiar, upgrade gradually
- **React Navigation 6** - Similar patterns to React Router
- **Material UI (MUI)** - Consistent Material Design components and theming
- **React Hook Form** - Same library you'd use in React web

### Backend & Database - Stick with Firebase
**Why Firebase makes sense for you:**
- **Firestore** - NoSQL you already know
- **Firebase Auth** - Authentication system you're familiar with
- **Cloud Functions** - Your Express/Node skills translate directly
- **Firebase Storage** - Simple file/image uploads
- **Real-time updates** - Built-in WebSocket-like functionality
- **Free tier** - Perfect for development and early testing

### Backend Alternative (If you want to learn)
- **Node.js + Express** (your existing skills) + **Firebase** for database
- Deploy to **Vercel** or **Railway** (beginner-friendly)

### Additional Services
- **Stripe** - Payment processing (has great React Native integration)
- **Expo Location** - Built-in GPS and mapping
- **Expo Camera** - Built-in photo capture
- **Firebase Cloud Messaging** - Push notifications (already in Firebase)
- **Expo Image Picker** - Built-in image selection

## 🗄 Database Schema Design

### Firestore Collections (Firebase Structure)

Since you're using Firebase, here's how to structure your data in Firestore:

```javascript
// Users Collection
users: {
  [userId]: {
    email: "user@example.com",
    fullName: "John Doe",
    phone: "+1234567890",
    avatarUrl: "https://...",
    userType: "cook" | "eater" | "both",
    isVerified: false,
    createdAt: timestamp,
    updatedAt: timestamp
  }
}

// Cook Profiles Collection
cookProfiles: {
  [userId]: {
    businessName: "Mom's Kitchen",
    description: "Authentic Italian cuisine...",
    specialties: ["italian", "vegetarian"],
    minAdvanceHours: 24,
    maxOrdersPerDay: 10,
    deliveryRadiusKm: 5.0,
    baseDeliveryFee: 3.50,
    isActive: true
  }
}

// Menu Items Collection
menuItems: {
  [menuItemId]: {
    cookId: "userId",
    title: "Homemade Lasagna",
    description: "Traditional recipe...",
    basePrice: 15.99,
    currency: "USD",
    category: "main-course",
    dietaryTags: ["vegetarian"],
    images: ["https://..."],
    ingredients: ["pasta", "cheese", "tomatoes"],
    allergens: ["dairy", "gluten"],
    minQuantity: 1,
    maxQuantity: 10,
    prepTimeHours: 2,
    isActive: true,
    createdAt: timestamp
  }
}

// Pricing Tiers Subcollection
menuItems/{menuItemId}/pricingTiers: {
  [tierId]: {
    minQuantity: 5,
    pricePerUnit: 14.99,
    isActive: true
  }
}

// Orders Collection
orders: {
  [orderId]: {
    buyerId: "userId",
    cookId: "userId",
    menuItemId: "menuItemId",
    quantity: 2,
    unitPrice: 15.99,
    totalAmount: 31.98,
    deliveryFee: 3.50,
    serviceFee: 2.50,
    status: "pending", // pending, accepted, preparing, ready, completed, cancelled
    requestedDate: "2024-01-15",
    requestedTime: "18:00",
    pickupAddress: "123 Main St",
    deliveryAddress: "456 Oak Ave",
    specialInstructions: "No onions please",
    createdAt: timestamp,
    updatedAt: timestamp
  }
}

// Reviews Collection
reviews: {
  [reviewId]: {
    orderId: "orderId",
    reviewerId: "userId",
    cookId: "userId",
    rating: 5,
    comment: "Amazing food!",
    createdAt: timestamp
  }
}

// User Locations Subcollection
users/{userId}/locations: {
  [locationId]: {
    label: "home",
    address: "123 Main St, City, State",
    latitude: 40.7128,
    longitude: -74.0060,
    isDefault: true
  }
}
```

## 🔐 Authentication & Authorization

### Authentication Flow (Firebase Auth)
1. **Email/Password** signup with Firebase Auth
2. **Google Sign-In** for easier onboarding (built into Expo)
3. **Phone verification** for cook accounts using Firebase Auth
4. **Profile completion** with role selection stored in Firestore

### Authorization with Firestore Security Rules
```javascript
// Firestore Security Rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can read/write their own profile
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }

    // Cook profiles - only the cook can modify
    match /cookProfiles/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == userId;
    }

    // Menu items - cooks can CRUD their own, everyone can read active ones
    match /menuItems/{menuItemId} {
      allow read: if resource.data.isActive == true;
      allow write: if request.auth != null && request.auth.uid == resource.data.cookId;
      allow create: if request.auth != null && request.auth.uid == request.resource.data.cookId;
    }

    // Orders - buyers and cooks can see their own orders
    match /orders/{orderId} {
      allow read, write: if request.auth != null &&
        (request.auth.uid == resource.data.buyerId || request.auth.uid == resource.data.cookId);
    }
  }
}
```

### Security Considerations
- **Firebase Auth** handles email verification automatically
- **Firestore Security Rules** protect user data
- **Cook verification** process using Firebase Storage for ID uploads
- **Stripe Connect** for secure payment processing

## 💳 Payment & Order Management

### Payment Flow
1. **Stripe Connect** for marketplace payments
2. **Escrow system** - hold funds until order completion
3. **Automatic payouts** to cooks after successful delivery
4. **Platform fee** (5-10%) deducted from cook earnings

### Order Lifecycle
```
Pending → Accepted → Preparing → Ready → Completed
    ↓
Cancelled (with refund policy)
```

### Key Features
- **Advance payment** required for all orders
- **Cancellation policy** (24h+ notice for full refund)
- **Dispute resolution** system with admin intervention
- **Automatic refunds** for cancelled orders

## 🔔 Push Notification & Scheduling System

### Push Notification Setup
**Required Packages** (install with npm/expo):
- `expo-notifications` - Core notification handling
- `expo-device` - Device detection
- `expo-constants` - App configuration
- `firebase/messaging` - Firebase Cloud Messaging (FCM)

**Configuration Needed**:
- Firebase Cloud Messaging (FCM) in Firebase Console
- VAPID key for web push notifications
- Push notification permissions (iOS/Android)
- Notification icons and sound files

### Real-time Push Notifications

#### Order Lifecycle Notifications:
- **Order Placed** → "New order received from [Buyer Name]" (to Cook)
- **Order Accepted** → "Maria Santos accepted your Chicken Mole order!" (to Buyer)
- **Order Declined** → "Your order was declined. Browse other options." (to Buyer)
- **Preparing** → "Your order is being prepared 👨‍🍳" (to Buyer)
- **Ready for Pickup** → "Food ready! Pickup at Maria's Kitchen" (to Buyer)
- **Completed** → "Order delivered! Please leave a review ⭐" (to Buyer)

#### Discovery & Engagement:
- **New Menu Items** → "Bangkok Betty added Pad Thai to the menu!" (to Followers)
- **Cook Available** → "Your favorite cook just opened slots for tomorrow" (to Followers)
- **Price Drops** → "Nonna Isabella reduced lasagna prices by 20%" (to Interested Users)
- **Last Chance** → "Only 2 slots left for tomorrow's orders" (to Users)

#### Reminders & Scheduling:
- **Pickup Reminders** → "Pickup in 30 minutes at [Address]" (to Buyer)
- **Prep Reminders** → "Don't forget to prep for tomorrow's 5 orders" (to Cook)
- **Review Requests** → "How was your meal? Rate Maria Santos" (to Buyer, 2 hours after pickup)
- **Reorder Suggestions** → "Craving Chicken Mole again? Reorder from Maria" (to Previous Buyers)

### Notification Categories & Targeting

#### By User Type:
**Cooks Receive**:
- New order alerts (immediate)
- Prep reminders (evening before)
- Capacity warnings (when approaching daily limit)
- Review notifications (when rated)
- Weekly earnings summary

**Buyers Receive**:
- Order status updates (real-time)
- Pickup reminders (30 min before)
- New menu notifications (from followed cooks)
- Special offers & discounts
- Recommendation notifications

#### By Urgency:
- **Critical** (Sound + Vibration): Order status changes, pickup reminders
- **Important** (Sound only): New orders, prep reminders
- **Info** (Silent): New menu items, weekly summaries

### Advanced Notification Features

#### Smart Scheduling:
- **Time-zone aware** delivery based on user location
- **Quiet hours** respect (no notifications 10PM-7AM)
- **Frequency capping** (max 3 promotional notifications per day)
- **User preferences** (can disable categories individually)

#### Personalization:
- **Location-based** → Only show nearby cook notifications
- **Dietary preferences** → Filter notifications by dietary tags
- **Order history** → Prioritize notifications from previously ordered cooks
- **Engagement patterns** → Send notifications when user is most active

#### Rich Notifications:
- **Images** → Show food photos in notification
- **Action buttons** → "Accept Order", "View Menu", "Rate Now"
- **Deep linking** → Direct navigation to specific screens
- **Custom sounds** → Different sounds for different notification types

### Technical Implementation

#### Notification Service Architecture:
```javascript
// Notification trigger types
- Local notifications (scheduled, immediate)
- Remote notifications (FCM from backend)
- Background notifications (when app is closed)
- Foreground notifications (when app is active)
```

#### Backend Integration:
**Cloud Functions** (Firebase) for automated notifications:
- Order status change triggers
- Scheduled reminder functions
- Batch notification sending
- Analytics and tracking

**Push Token Management**:
- Store user push tokens in Firestore
- Update tokens on app login/logout
- Handle token refresh automatically
- Remove tokens when user uninstalls

### Notification Analytics & Optimization

#### Tracking Metrics:
- **Delivery rates** → How many notifications reach users
- **Open rates** → How many users tap notifications
- **Conversion rates** → Notifications leading to orders
- **Unsubscribe rates** → Users disabling notifications

#### A/B Testing:
- **Message content** → Test different notification copy
- **Send timing** → Optimize delivery times
- **Frequency** → Find optimal notification cadence
- **Personalization** → Test targeted vs. general notifications

### Notification Permission Strategy

#### Progressive Permission Requests:
1. **Onboarding** → Explain notification benefits
2. **First Order** → "Get updates on your order status?"
3. **Cook Setup** → "Receive new order notifications?"
4. **Feature Discovery** → "Get notified about new menu items?"

#### Fallback Options:
- **In-app notifications** for users who decline push
- **Email notifications** as secondary channel
- **SMS notifications** for critical updates (optional)

### Scheduling Features
- **Calendar integration** for cooks to manage availability
- **Batch cooking** support for multiple orders
- **Lead time management** per cook and menu item
- **Capacity management** to prevent overselling
- **Smart scheduling** algorithms for optimal delivery times

### Notification Best Practices

#### Content Guidelines:
- **Personalized** → Use buyer/cook names
- **Actionable** → Clear next steps
- **Timely** → Sent at relevant moments
- **Valuable** → Provide useful information
- **Concise** → Under 50 characters for titles

#### Frequency Management:
- **Immediate**: Order status updates
- **Daily**: Max 3 promotional notifications
- **Weekly**: Summary and recommendation notifications
- **Monthly**: App feature updates and tips

---

**Implementation Priority**: Start with basic order status notifications, then add discovery and reminder notifications as the app grows.

## 📱 App Architecture

### Screen Structure
```
Auth Flow
├── Welcome/Onboarding
├── Login/Signup
└── Profile Setup

Main App (Tab Navigation)
├── Discover
│   ├── Map View
│   ├── List View
│   └── Filters
├── Orders
│   ├── Active Orders
│   ├── Order History
│   └── Order Details
├── Cook Dashboard (Cook Users)
│   ├── Menu Management
│   ├── Order Management
│   ├── Availability Calendar
│   └── Analytics
└── Profile
    ├── Account Settings
    ├── Payment Methods
    ├── Addresses
    └── Reviews
```

### Key Components
- **Menu Item Card** with photos, pricing, cook info
- **Order Tracking** component with real-time updates
- **Chat Interface** for cook-buyer communication
- **Rating/Review** system with photos
- **Map Integration** for location-based discovery

## 🚀 Development Phases

### Phase 1: MVP (8-12 weeks)
- **User authentication** and basic profiles
- **Menu item CRUD** for cooks
- **Basic ordering** system
- **Payment integration** (Stripe)
- **Core UI/UX** implementation

### Phase 2: Enhanced Features (6-8 weeks)
- **Advanced search** and filtering
- **Real-time notifications**
- **Review and rating** system
- **Map-based discovery**
- **Order tracking** improvements

### Phase 3: Community Features (4-6 weeks)
- **Cook following** system
- **Favorite menus** and repeat ordering
- **Social sharing** features
- **Referral program**
- **Advanced analytics** for cooks

### Phase 4: Scale & Optimize (Ongoing)
- **Performance optimization**
- **Advanced matching algorithms**
- **Business intelligence** tools
- **Expansion features** (catering, events)

## ⚖️ Legal & Safety Considerations

### Food Safety
- **Cook education** on food safety practices
- **Allergen disclosure** requirements
- **Insurance recommendations** for cooks
- **Health department guidelines** by location

### Legal Compliance
- **Terms of service** covering liability
- **Local licensing** requirements research
- **Tax reporting** tools for cook earnings
- **GDPR/CCPA** compliance for user data

### Risk Mitigation
- **User verification** processes
- **Insurance partnerships** for protection
- **Incident reporting** system
- **Emergency contact** features

## 💰 Monetization Strategy

### Revenue Streams
1. **Platform commission** (5-10% per transaction)
2. **Premium cook subscriptions** (enhanced features)
3. **Advertising** (promoted menu items)
4. **Processing fees** (small markup on payment processing)

### Growth Strategy
- **Local market penetration** before geographic expansion
- **Cook incentive programs** for early adopters
- **Referral bonuses** for both cooks and eaters
- **Community partnerships** (farmers markets, food blogs)

## 📊 Success Metrics

### Core KPIs
- **Monthly Active Users** (both cook and eater)
- **Order completion rate**
- **Average order value**
- **Cook retention rate**
- **Customer satisfaction** (reviews/ratings)

### Technical Metrics
- **App performance** (load times, crash rates)
- **Payment success rate**
- **Notification delivery rate**
- **Search/discovery effectiveness**

## 🎯 Getting Started (Step-by-Step for Beginners)

### Phase 0: Setup & Learning (1-2 weeks)

#### 1. Install Development Tools
```bash
# Install Node.js (you probably have this)
# Install Expo CLI
npm install -g @expo/cli

# Create your first React Native app
npx create-expo-app HomeCookConnect
cd HomeCookConnect
```

#### 2. Set up Firebase Project
1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Create new project: "HomeCookConnect"
3. Enable Authentication (Email/Password, Google)
4. Create Firestore database
5. Get your Firebase config object

#### 3. Install Essential Packages
```bash
# Firebase
npm install firebase

# Navigation
npm install @react-navigation/native @react-navigation/bottom-tabs @react-navigation/stack
npx expo install react-native-screens react-native-safe-area-context

# UI Components
npm install @mui/material @emotion/react @emotion/styled
npm install @mui/icons-material
npx expo install react-native-svg

# Forms
npm install react-hook-form

# Image & Location
npx expo install expo-image-picker expo-location expo-camera
```

#### 4. Learning Resources
- **React Native basics**: [Expo Documentation](https://docs.expo.dev/)
- **Firebase with React Native**: [Firebase Docs](https://firebase.google.com/docs/web/setup)
- **React Navigation**: [Navigation Docs](https://reactnavigation.org/docs/getting-started)

### Phase 1: MVP Development (6-8 weeks)

#### Week 1-2: Authentication & User Profiles
- Set up Firebase config
- Build login/signup screens
- Create user profile setup
- Implement role selection (cook vs eater)

#### Week 3-4: Menu Management (Cook Side)
- Create menu item form
- Implement image upload to Firebase Storage
- Build menu item list/edit screens
- Add pricing tiers functionality

#### Week 5-6: Discovery & Ordering (Eater Side)
- Build menu discovery screen
- Implement basic search/filtering
- Create order placement flow
- Add order confirmation

#### Week 7-8: Order Management & Polish
- Build order tracking for both users
- Implement order status updates
- Add basic notifications
- Polish UI/UX

### Simple First Features to Build
1. **User Registration** - Start here, it's just forms
2. **Menu Item Creation** - Practice with Firebase CRUD
3. **Menu Discovery** - Learn Firebase queries
4. **Basic Ordering** - Connect the pieces

### 🚀 Next Development Phases

**Phase 2: Enhanced Features (4-6 weeks)**
- Stripe payment integration
- Real-time order updates
- Review system
- Location-based search

**Phase 3: Community Features (4-6 weeks)**
- Cook following
- Advanced notifications
- Social features
- Analytics

## 💡 Beginner Tips

### Start Simple
- Build one screen at a time
- Get Firebase working first before adding complexity
- Use Expo's built-in components before custom ones
- Start with functional components and useState

### Debugging Help
- Use Expo Go app for testing on your phone
- Expo DevTools for debugging
- Firebase Console to see your data
- React Native Debugger (optional but helpful)

### Common Gotchas
- Firebase security rules start restrictive
- Expo Location needs permissions setup
- Image uploads need proper error handling
- Always test on actual device, not just simulator

## 🍽️ Mock Data Structure

### Home Cook Profiles (10 Different Cooks)

#### 1. **Maria Santos** - ⭐ 4.8/5 (127 reviews)
**Specialty**: Authentic Mexican & Latin American
**Bio**: "Third-generation cook bringing abuela's secret recipes to your table. 15 years perfecting traditional moles, tamales, and comfort foods that warm the soul. Every dish tells a story of family tradition."
**Menu Items (8)**:
- Chicken Mole Poblano ($18)
- Beef Barbacoa Tacos (6 pack) ($15)
- Homemade Tamales (4 pack) ($12)
- Pozole Rojo Bowl ($14)
- Elote Street Corn (2 pieces) ($8)
- Tres Leches Cake Slice ($6)
- Fresh Guacamole & Chips ($9)
- Agua Fresca (32oz) ($5)

#### 2. **Chef Antoine Dubois** - ⭐ 4.9/5 (89 reviews)
**Specialty**: Classical French Cuisine
**Bio**: "Former sous chef at Le Bernardin, now crafting restaurant-quality French classics from my home kitchen. 20 years mastering the art of French cooking, bringing Michelin-level techniques to everyday dining."
**Menu Items (12)**:
- Coq au Vin ($24)
- Beef Bourguignon ($26)
- Duck Confit with Orange Glaze ($28)
- Bouillabaisse ($22)
- Ratatouille Niçoise ($16)
- French Onion Soup ($12)
- Crème Brûlée ($8)
- Tarte Tatin ($10)
- Escargots (6 pieces) ($14)
- Quiche Lorraine ($18)
- Croissants (4 pack) ($12)
- Madeleines (8 pack) ($9)

#### 3. **Nonna Isabella** - ⭐ 4.7/5 (203 reviews)
**Specialty**: Traditional Italian
**Bio**: "Born in Tuscany, cooking for 50+ years. My pasta is made fresh daily, and my sauce recipe has been in the family for four generations. Bringing authentic Italian comfort to every plate."
**Menu Items (10)**:
- Homemade Lasagna ($20)
- Chicken Parmigiana ($18)
- Osso Buco ($25)
- Fresh Fettuccine Alfredo ($16)
- Eggplant Parmigiana ($17)
- Tiramisu Cup ($7)
- Bruschetta (4 pieces) ($10)
- Minestrone Soup ($11)
- Risotto Milanese ($19)
- Cannoli (2 pieces) ($8)

#### 4. **Bangkok Betty** - ⭐ 4.6/5 (156 reviews)
**Specialty**: Authentic Thai Street Food
**Bio**: "Street food chef from Bangkok bringing authentic flavors to your doorstep. 12 years perfecting the balance of sweet, sour, salty, and spicy that makes Thai cuisine unforgettable."
**Menu Items (14)**:
- Pad Thai Shrimp ($15)
- Green Curry Chicken ($16)
- Tom Yum Soup ($12)
- Massaman Beef Curry ($18)
- Mango Sticky Rice ($8)
- Papaya Salad ($10)
- Thai Basil Stir Fry ($14)
- Coconut Soup (Tom Kha) ($13)
- Pad See Ew ($15)
- Thai Fried Rice ($12)
- Spring Rolls (6 pack) ($11)
- Larb Gai Salad ($13)
- Red Curry Vegetables ($14)
- Thai Iced Tea ($4)

#### 5. **Sarah Green** - ⭐ 4.4/5 (78 reviews)
**Specialty**: Plant-Based & Healthy Options
**Bio**: "Nutritionist turned chef, creating delicious plant-based meals that don't compromise on flavor. 8 years developing recipes that make healthy eating exciting and satisfying for everyone."
**Menu Items (11)**:
- Buddha Bowl with Tahini Dressing ($14)
- Vegan Mac & Cheese ($13)
- Quinoa Stuffed Bell Peppers ($15)
- Raw Zucchini Noodles ($12)
- Chickpea Curry ($13)
- Acai Bowl ($10)
- Overnight Oats Cup ($7)
- Kale Caesar Salad ($11)
- Veggie Burger with Sweet Potato Fries ($16)
- Chia Pudding Parfait ($8)
- Green Smoothie Bowl ($9)

#### 6. **BBQ Bob Wilson** - ⭐ 4.5/5 (134 reviews)
**Specialty**: Southern BBQ & Comfort Food
**Bio**: "Pitmaster with 25 years smoking meats low and slow. Competition BBQ winner bringing authentic Southern flavors and time-honored techniques to every rack of ribs and brisket."
**Menu Items (9)**:
- Smoked Brisket Platter ($22)
- Full Rack Baby Back Ribs ($24)
- Pulled Pork Sandwich ($14)
- BBQ Chicken Half ($16)
- Mac and Cheese Side ($8)
- Coleslaw ($6)
- Cornbread (4 pieces) ($7)
- Baked Beans ($7)
- Peach Cobbler ($8)

#### 7. **Akiko Tanaka** - ⭐ 4.8/5 (167 reviews)
**Specialty**: Japanese Home Cooking
**Bio**: "Tokyo native sharing the comfort foods of Japan. 18 years perfecting the art of simple, seasonal ingredients prepared with precision and care. Every meal is a meditation on flavor."
**Menu Items (13)**:
- Chicken Teriyaki Bento ($17)
- Ramen Bowl (Tonkotsu) ($16)
- Sushi Roll Set (8 pieces) ($19)
- Chicken Katsu Curry ($18)
- Miso Soup ($6)
- Gyoza (6 pieces) ($10)
- Onigiri (2 pieces) ($8)
- Tempura Vegetables ($14)
- Yakitori Skewers (4 pieces) ($12)
- Mochi Ice Cream (3 pieces) ($9)
- Dorayaki Pancakes (2 pieces) ($7)
- Matcha Cheesecake Slice ($8)
- Japanese Potato Salad ($7)

#### 8. **Mom's Kitchen (Linda Johnson)** - ⭐ 2.8/5 (45 reviews)
**Specialty**: American Comfort Food
**Bio**: "Home cook trying to make ends meet with family recipes. Still learning the ropes of cooking for others, but my heart is in every dish. Your patience and feedback help me grow!"
**Menu Items (6)**:
- Meatloaf with Mashed Potatoes ($13)
- Fried Chicken (3 pieces) ($12)
- Tuna Casserole ($10)
- Chocolate Chip Cookies (6 pack) ($6)
- Beef Stew ($11)
- Apple Pie Slice ($5)

#### 9. **Global Fusion Felix** - ⭐ 4.2/5 (92 reviews)
**Specialty**: International Fusion
**Bio**: "World traveler turned chef, blending flavors from 30+ countries into unique fusion creations. 10 years experimenting with cross-cultural cooking that surprises and delights adventurous eaters."
**Menu Items (15)**:
- Korean BBQ Tacos (3 pack) ($16)
- Butter Chicken Pizza ($18)
- Mediterranean Sushi Bowl ($15)
- Thai Curry Risotto ($17)
- Moroccan Lamb Sliders (3 pack) ($19)
- Vietnamese Pho Ramen ($14)
- Indian Spiced Mac & Cheese ($13)
- Greek Gyro Bowl ($16)
- Peruvian Ceviche Tostadas ($17)
- Jamaican Jerk Stir Fry ($15)
- Middle Eastern Flatbread Pizza ($14)
- Brazilian Açaí Tacos (2 pieces) ($12)
- Chinese Five-Spice Lamb Chops ($24)
- Cuban Sandwich Spring Rolls ($11)
- Chai Spiced Crème Brûlée ($8)

#### 10. **Priya Sharma** - ⭐ 4.3/5 (119 reviews)
**Specialty**: Regional Indian Cuisine
**Bio**: "Born in Mumbai, trained in classical Indian cooking techniques. 14 years mastering regional specialties from North to South India. Each spice blend is carefully crafted to transport you to India."
**Menu Items (12)**:
- Chicken Tikka Masala ($16)
- Lamb Biryani ($19)
- Palak Paneer ($14)
- Dal Makhani ($12)
- Samosas (4 pieces) ($8)
- Tandoori Chicken Half ($17)
- Butter Naan (2 pieces) ($6)
- Mango Lassi ($5)
- Gulab Jamun (3 pieces) ($7)
- Aloo Gobi ($13)
- Chole Bhature ($15)
- Ras Malai (2 pieces) ($8)

### Rating Distribution Strategy
- **Poorly Rated (1 cook)**: Mom's Kitchen - 2.8/5 stars (new cook, still learning)
- **Average Rated (2 cooks)**: BBQ Bob (4.2/5), Global Fusion Felix (4.2/5)
- **Well Rated (7 cooks)**: All others ranging from 4.3/5 to 4.9/5

### Image Requirements
- Each cook needs a profile photo (portrait style)
- Each menu item needs 1-3 high-quality food photos
- Total images needed: ~120-150 royalty-free food images
- Style should be consistent - appetizing, well-lit, professional food photography

---

*This beginner-friendly plan leverages your existing React/Firebase knowledge while introducing mobile development concepts gradually. Start with the setup phase and build one feature at a time!*