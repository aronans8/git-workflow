# GAMIFICATION SYSTEM - SƠ ĐỒ TỔNG THỂ

## OVERVIEW
Hệ thống gamification xử lý từ hoạt động của user (bet/deposit/login) đến việc nhận reward qua các bước:

```
USER ACTIVITY → TRACKING → CALCULATION → COMPLETION → REWARD
```

---

## SƠ ĐỒ LUỒNG TỔNG THỂ

```mermaid
graph TD
    %% USER ACTIVITIES
    A[User Activities] --> A1[Bet Activity]
    A --> A2[Deposit Activity] 
    A --> A3[Login Activity]
    
    %% EXTERNAL SYSTEMS
    A1 --> B1[s8_gateway - Kafka Producer]
    A2 --> B2[Payment Service - Kafka Producer]
    A3 --> B3[Auth Service - Kafka Producer]
    
    %% KAFKA TOPICS
    B1 --> K1[gamification.bet.activity]
    B2 --> K2[gamification.deposit.activity]
    B3 --> K3[gamification.login.activity]
    
    %% ACTIVITY TRACKING
    K1 --> C[ActivityTrackingService - Kafka Consumer]
    K2 --> C
    K3 --> C
    
    %% MISSION PROCESSING
    C --> D1[Find Applicable Missions]
    D1 --> D2[Calculate Progress]
    D2 --> D3[Update MissionRecord]
    
    %% COMPLETION CHECK
    D3 --> E1{Mission Completed?}
    E1 -->|Yes| E2[Award Reward]
    E1 -->|No| E3[Continue Tracking]
    
    %% REWARD SYSTEM
    E2 --> F1[Add to ClaimedMission]
    F1 --> F2[Distribute Reward]
    F2 --> F3[Notify User]
    
    %% STREAK HANDLING
    D3 --> G1[Update Streaks]
    G1 --> G2[Reset Daily/Weekly Missions]
    
    %% DATABASE
    D3 --> H[(MongoDB mission_record)]
    F1 --> H
    
    style A fill:#e1f5fe
    style C fill:#f3e5f5
    style E2 fill:#e8f5e8
    style F2 fill:#fff3e0
```

---

## KIẾN TRÚC HỆ THỐNG

### 1. **EXTERNAL INTEGRATION LAYER**
```java
// Kafka Topics for external systems integration
Topic: "gamification.bet.activity"      // từ s8_gateway
Topic: "gamification.deposit.activity"  // từ payment service  
Topic: "gamification.login.activity"    // từ auth service

// Kafka Consumers
@KafkaListener(topics = "gamification.bet.activity")
@KafkaListener(topics = "gamification.deposit.activity")
@KafkaListener(topics = "gamification.login.activity")
```

### 2. **ACTIVITY PROCESSING LAYER**
```java
@Service
@Component
public class ActivityTrackingService {
    
    @KafkaListener(topics = "gamification.bet.activity")
    public void processBetActivity(BetActivityEvent event) {
        // Xử lý bet events từ Kafka
    }
    
    @KafkaListener(topics = "gamification.deposit.activity") 
    public void processDepositActivity(DepositActivityEvent event) {
        // Xử lý deposit events từ Kafka
    }
    
    @KafkaListener(topics = "gamification.login.activity")
    public void processLoginActivity(LoginActivityEvent event) {
        // Xử lý login events từ Kafka
    }
}

### 3. **MISSION CALCULATION LAYER**
```java
calculateNewProgress() {
    BET_AMOUNT     → Add to current amount
    BET_COUNT      → Increment count
    WIN_STREAK     → Handle streak logic
    DEPOSIT_AMOUNT → Add to deposit total
    DAILY_LOGIN    → Increment login count
}
```

### 4. **DATA PERSISTENCE LAYER**
```java
MissionRecord {
    userId           // User identifier
    missionProgress  // List<MissionProgress>
    claimedMission   // List<ClaimedMission>
}
```

---

## QUY TRÌNH XỬ LÝ CHI TIẾT

### **STEP 1: User Activity Occurs**
```javascript
// Ví dụ: User đặt cược 100k
{
  "userId": "user123",
  "betAmount": 100000,
  "gameType": "sports",
  "betTime": "2025-09-01T10:30:00"
}
```

### **STEP 2: External System Sends Kafka Message**
```java
// s8_gateway publish kafka message
Topic: "gamification.bet.activity"
{
  "userId": "user123",
  "betAmount": 100000,
  "gameType": "sports",
  "betTime": "2025-09-01T10:30:00",
  "betId": "bet_123456"
}

// payment service publish kafka message  
Topic: "gamification.deposit.activity"
{
  "userId": "user123",
  "depositAmount": 500000,
  "paymentMethod": "bank_transfer",
  "transactionId": "txn_789012"
}
```

### **STEP 3: Find Applicable Missions**
```java
findApplicableMissionsForBet(event) {
    return [
        "daily_bet_amount_100k",    // BET_AMOUNT: target 100k
        "bet_count_daily_10",       // BET_COUNT: target 10 bets  
        "win_streak_5"              // WIN_STREAK: target 5 wins
    ];
}
```

### **STEP 4: Calculate Progress**
```java
// Mission: daily_bet_amount_100k (target: 100k)
currentProgress = 50k    // đã bet 50k trước đó
activityValue = 100k     // bet hiện tại
newProgress = 50k + 100k = 150k  // COMPLETED!

// Mission: bet_count_daily_10 (target: 10)
currentProgress = 8      // đã bet 8 lần
newProgress = 8 + 1 = 9  // chưa complete
```

### **STEP 5: Update Database**
```java
MissionProgress progress = {
    missionCode: "daily_bet_amount_100k",
    currentProgress: 150000,        // 150k
    target: 100000,                 // 100k
    status: COMPLETED,              // Completed
    completedAt: "2025-09-01T10:30:00"
}
```

### **STEP 6: Award Reward** 
```java
// Mission completed → Claim reward
ClaimedMission claimed = {
    missionCode: "daily_bet_amount_100k",
    claimedAt: "2025-09-01T10:30:00",
    claimedReward: {
        type: "COINS",
        amount: 5000,               // 5k coins
        description: "Daily bet bonus"
    }
}
```

---

## MISSION TYPES VÀ CALCULATION

| Mission Type | Activity Input | Calculation Logic | Example |
|--------------|----------------|-------------------|---------|
| **BET_AMOUNT** | Bet amount | `current + amount` | 50k + 100k = 150k |
| **BET_COUNT** | Any bet | `current + 1` | 8 + 1 = 9 bets |
| **WIN_STREAK** | Win/Loss | Win: `+1`, Loss: `reset to 0` | 3 wins → 4 wins |
| **DEPOSIT_AMOUNT** | Deposit amount | `current + amount` | 200k + 500k = 700k |
| **DAILY_LOGIN** | Login event | `current + 1` | 5 days → 6 days |
| **REFER_FRIEND** | Referral | `current + 1` | 2 refs → 3 refs |

---

## REWARD DISTRIBUTION FLOW

```mermaid
graph LR
    A[Mission Completed] --> B[Calculate Reward]
    B --> C[Award Coins/Tokens]
    C --> D[Push Notification]
    D --> E[Update User Stats]
    E --> F[Achievement Unlocked]
```

### **Reward Types:**
- **Coins**: Virtual currency
- **Tokens**: Premium currency  
- **Items**: In-game items
- **Badges**: Achievement badges
- **Points**: Loyalty points
- **Bonuses**: Free bets/spins

---

## MULTI-EVENT SUPPORT

```java
// User có thể tham gia nhiều events đồng thời
EventProgressSummary {
    "euro_cup_2024": {
        totalMissions: 10,
        completedMissions: 6,      // 60% complete
        overallPercentage: 85.5%   // based on progress values
    },
    "world_cup_2024": {
        totalMissions: 8,
        completedMissions: 3,      // 37.5% complete  
        overallPercentage: 42.1%
    }
}
```

---

## DATA LIFECYCLE

### **Daily Reset:**
```java
// Reset daily missions at midnight
resetDailyMissions() {
    - daily_login_bonus: reset to 0
    - daily_bet_amount: reset to 0
    - daily_deposit: reset to 0
}
```

### **Streak Maintenance:**
```java
// Check streaks and reset if broken
maintainStreaks() {
    if (noLoginToday) loginStreak = 0;
    if (noDepositToday) depositStreak = 0;
    if (betLoss) winStreak = 0;
}
```

### **Archive Old Data:**
```java
// Archive completed events after 6 months
archiveExpiredData() {
    - Move old MissionProgress to archive
    - Keep ClaimedMission for history
    - Cleanup expired progress data
}
```

---

## TỔNG KẾT

**INPUT**: User activities (bet/deposit/login)  
**PROCESSING**: Mission matching → Progress calculation → Completion check  
**OUTPUT**: Rewards (coins/tokens/items) + Notifications

**Key Components:**
1. **ActivityTrackingService** - Process external events
2. **MissionRecord** - Track user progress & rewards  
3. **MissionProgress** - Individual mission state
4. **ClaimedMission** - Reward history
5. **External APIs** - Integration points

**Result**: Complete gamification ecosystem từ user action đến reward distribution!
