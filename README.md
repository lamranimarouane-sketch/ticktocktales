# 🕐 TickTock Tales — Flutter Clock Learning App

## Architecture Overview

This app follows **Clean Architecture** with **Riverpod** for state management.

```
lib/
├── main.dart
├── app/
│   ├── app.dart                    # MaterialApp + routing
│   ├── theme/
│   │   ├── app_theme.dart
│   │   └── app_colors.dart
│   └── routes/
│       └── app_router.dart         # GoRouter setup
├── core/
│   ├── constants/
│   │   └── app_constants.dart
│   ├── utils/
│   │   ├── time_utils.dart         # Time generation, formatting
│   │   └── sound_utils.dart        # Sound effect helpers
│   └── l10n/                       # Localization
│       ├── app_en.arb
│       ├── app_fr.arb
│       ├── app_es.arb
│       └── app_ar.arb
├── features/
│   ├── home/
│   │   ├── presentation/
│   │   │   ├── home_screen.dart
│   │   │   └── widgets/
│   │   │       └── level_card.dart
│   ├── clock/
│   │   ├── domain/
│   │   │   └── clock_time.dart     # Time value object
│   │   ├── data/
│   │   │   └── clock_repository.dart
│   │   └── presentation/
│   │       ├── widgets/
│   │       │   ├── analog_clock.dart      # Main clock widget
│   │       │   ├── clock_hand.dart        # Individual hand painter
│   │       │   └── clock_face_painter.dart
│   │       └── clock_screen.dart
│   ├── quiz/
│   │   ├── domain/
│   │   │   ├── quiz_question.dart
│   │   │   └── quiz_result.dart
│   │   ├── data/
│   │   │   └── question_generator.dart
│   │   ├── presentation/
│   │   │   ├── quiz_screen.dart
│   │   │   ├── result_screen.dart
│   │   │   └── widgets/
│   │   │       ├── multiple_choice_widget.dart
│   │   │       ├── confetti_overlay.dart
│   │   │       └── progress_bar.dart
│   │   └── providers/
│   │       └── quiz_provider.dart
│   ├── gamification/
│   │   ├── domain/
│   │   │   ├── badge.dart
│   │   │   └── player_stats.dart
│   │   ├── data/
│   │   │   └── stats_repository.dart   # SharedPreferences
│   │   └── providers/
│   │       └── stats_provider.dart
│   └── parent_mode/
│       ├── presentation/
│       │   └── parent_dashboard.dart
│       └── providers/
│           └── parent_provider.dart
└── shared/
    └── widgets/
        ├── kid_button.dart
        ├── star_rating.dart
        └── reward_animation.dart
```

---

## 🚀 Quick Start

### Prerequisites
```bash
flutter --version  # Requires Flutter 3.16+
```

### Install & Run
```bash
flutter pub get
flutter run
# For specific platform:
flutter run -d android
flutter run -d ios
```

### Generate Localization Files
```bash
flutter gen-l10n
```

---

## 📦 pubspec.yaml Dependencies

```yaml
name: ticktock_tales
description: Learn to tell time — for kids!
version: 1.0.0+1

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter

  # State Management
  flutter_riverpod: ^2.4.9
  riverpod_annotation: ^2.3.3

  # Navigation
  go_router: ^13.0.0

  # Local Storage
  shared_preferences: ^2.2.2

  # Audio
  audioplayers: ^5.2.1

  # Animations
  confetti: ^0.7.0
  lottie: ^3.0.0
  flutter_animate: ^4.3.0

  # Utilities
  equatable: ^2.0.5
  uuid: ^4.2.1

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.4.7
  riverpod_generator: ^2.3.9
  flutter_lints: ^3.0.0

flutter:
  uses-material-design: true
  generate: true  # enables l10n generation
  
  assets:
    - assets/sounds/
    - assets/images/badges/
    - assets/lottie/

  fonts:
    - family: Fredoka
      fonts:
        - asset: assets/fonts/Fredoka-Regular.ttf
        - asset: assets/fonts/Fredoka-Bold.ttf
          weight: 700
```

---

## 🎨 Core Code — Key Files

### `lib/main.dart`
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:shared_preferences/shared_preferences.dart';
import 'app/app.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final prefs = await SharedPreferences.getInstance();
  
  runApp(
    ProviderScope(
      overrides: [
        sharedPreferencesProvider.overrideWithValue(prefs),
      ],
      child: const TickTockApp(),
    ),
  );
}
```

### `lib/app/app.dart`
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:go_router/go_router.dart';
import '../core/l10n/app_localizations.dart';
import 'routes/app_router.dart';
import 'theme/app_theme.dart';

class TickTockApp extends ConsumerWidget {
  const TickTockApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final locale = ref.watch(localeProvider);
    final router = ref.watch(routerProvider);

    return MaterialApp.router(
      title: 'TickTock Tales',
      theme: AppTheme.lightTheme,
      routerConfig: router,
      locale: locale,
      supportedLocales: AppLocalizations.supportedLocales,
      localizationsDelegates: const [
        AppLocalizations.delegate,
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
      ],
      builder: (context, child) {
        // Enforce LTR/RTL based on locale
        return Directionality(
          textDirection: locale.languageCode == 'ar'
              ? TextDirection.rtl
              : TextDirection.ltr,
          child: child!,
        );
      },
    );
  }
}
```

---

### `lib/features/clock/presentation/widgets/analog_clock.dart`
```dart
import 'package:flutter/material.dart';
import 'dart:math' as math;

/// Interactive analog clock with draggable hands.
class AnalogClock extends StatefulWidget {
  final TimeOfDay initialTime;
  final bool interactive;
  final bool showDigital;
  final Function(TimeOfDay)? onTimeChanged;

  const AnalogClock({
    super.key,
    required this.initialTime,
    this.interactive = false,
    this.showDigital = true,
    this.onTimeChanged,
  });

  @override
  State<AnalogClock> createState() => _AnalogClockState();
}

class _AnalogClockState extends State<AnalogClock>
    with TickerProviderStateMixin {
  late TimeOfDay _currentTime;
  late AnimationController _tickController;

  @override
  void initState() {
    super.initState();
    _currentTime = widget.initialTime;
    _tickController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    )..repeat();
  }

  @override
  void dispose() {
    _tickController.dispose();
    super.dispose();
  }

  void _handlePanUpdate(DragUpdateDetails details, bool isMinuteHand) {
    if (!widget.interactive) return;
    final center = Offset(150, 150); // half of clock size
    final angle = math.atan2(
      details.localPosition.dy - center.dy,
      details.localPosition.dx - center.dx,
    );
    
    setState(() {
      if (isMinuteHand) {
        final minutes = ((angle + math.pi / 2) / (2 * math.pi) * 60)
            .round()
            .clamp(0, 59);
        _currentTime = TimeOfDay(hour: _currentTime.hour, minute: minutes.toInt());
      } else {
        final hours = ((angle + math.pi / 2) / (2 * math.pi) * 12)
            .round()
            .clamp(0, 11);
        _currentTime = TimeOfDay(hour: hours.toInt(), minute: _currentTime.minute);
      }
      widget.onTimeChanged?.call(_currentTime);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        GestureDetector(
          onPanUpdate: (d) => _handlePanUpdate(d, true),
          child: CustomPaint(
            size: const Size(300, 300),
            painter: ClockFacePainter(time: _currentTime),
          ),
        ),
        if (widget.showDigital)
          Padding(
            padding: const EdgeInsets.only(top: 16),
            child: Text(
              '${_currentTime.hour.toString().padLeft(2, '0')}:'
              '${_currentTime.minute.toString().padLeft(2, '0')}',
              style: const TextStyle(
                fontFamily: 'Fredoka',
                fontSize: 36,
                fontWeight: FontWeight.bold,
                letterSpacing: 4,
              ),
            ),
          ),
      ],
    );
  }
}

class ClockFacePainter extends CustomPainter {
  final TimeOfDay time;

  const ClockFacePainter({required this.time});

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2;

    _drawFace(canvas, center, radius);
    _drawNumbers(canvas, center, radius);
    _drawHourHand(canvas, center, radius);
    _drawMinuteHand(canvas, center, radius);
    _drawCenter(canvas, center);
  }

  void _drawFace(Canvas canvas, Offset center, double radius) {
    // Outer shadow
    final shadowPaint = Paint()
      ..color = Colors.black.withOpacity(0.15)
      ..maskFilter = const MaskFilter.blur(BlurStyle.normal, 8);
    canvas.drawCircle(center + const Offset(4, 4), radius, shadowPaint);

    // Face gradient
    final facePaint = Paint()
      ..shader = RadialGradient(
        colors: [Colors.white, const Color(0xFFF0F4FF)],
      ).createShader(Rect.fromCircle(center: center, radius: radius));
    canvas.drawCircle(center, radius, facePaint);

    // Rim
    final rimPaint = Paint()
      ..color = const Color(0xFF5C6BC0)
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8;
    canvas.drawCircle(center, radius - 4, rimPaint);

    // Hour tick marks
    for (int i = 0; i < 12; i++) {
      final angle = (i * 30 - 90) * math.pi / 180;
      final isHour = true;
      final tickPaint = Paint()
        ..color = const Color(0xFF3949AB)
        ..strokeWidth = i % 3 == 0 ? 4 : 2
        ..strokeCap = StrokeCap.round;
      final outerR = radius - 12;
      final innerR = radius - (i % 3 == 0 ? 28 : 20);
      canvas.drawLine(
        center + Offset(math.cos(angle) * innerR, math.sin(angle) * innerR),
        center + Offset(math.cos(angle) * outerR, math.sin(angle) * outerR),
        tickPaint,
      );
    }
  }

  void _drawNumbers(Canvas canvas, Offset center, double radius) {
    final textPainter = TextPainter(textDirection: TextDirection.ltr);
    for (int i = 1; i <= 12; i++) {
      final angle = (i * 30 - 90) * math.pi / 180;
      final offset = center + Offset(
        math.cos(angle) * (radius - 48),
        math.sin(angle) * (radius - 48),
      );
      textPainter.text = TextSpan(
        text: '$i',
        style: const TextStyle(
          fontFamily: 'Fredoka',
          color: Color(0xFF1A237E),
          fontSize: 22,
          fontWeight: FontWeight.bold,
        ),
      );
      textPainter.layout();
      textPainter.paint(
        canvas,
        offset - Offset(textPainter.width / 2, textPainter.height / 2),
      );
    }
  }

  void _drawHourHand(Canvas canvas, Offset center, double radius) {
    final angle = ((time.hour % 12) * 30 + time.minute * 0.5 - 90) * math.pi / 180;
    final handLength = radius * 0.55;
    final paint = Paint()
      ..color = const Color(0xFF1A237E)
      ..strokeWidth = 8
      ..strokeCap = StrokeCap.round;
    canvas.drawLine(
      center,
      center + Offset(math.cos(angle) * handLength, math.sin(angle) * handLength),
      paint,
    );
  }

  void _drawMinuteHand(Canvas canvas, Offset center, double radius) {
    final angle = (time.minute * 6 - 90) * math.pi / 180;
    final handLength = radius * 0.75;
    final paint = Paint()
      ..color = const Color(0xFF5C6BC0)
      ..strokeWidth = 5
      ..strokeCap = StrokeCap.round;
    canvas.drawLine(
      center,
      center + Offset(math.cos(angle) * handLength, math.sin(angle) * handLength),
      paint,
    );
  }

  void _drawCenter(Canvas canvas, Offset center) {
    canvas.drawCircle(center, 10, Paint()..color = const Color(0xFFFF6B6B));
    canvas.drawCircle(center, 5, Paint()..color = Colors.white);
  }

  @override
  bool shouldRepaint(ClockFacePainter oldDelegate) =>
      oldDelegate.time != time;
}
```

---

### `lib/features/quiz/providers/quiz_provider.dart`
```dart
import 'dart:math';
import 'package:flutter/material.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../domain/quiz_question.dart';
import '../domain/quiz_result.dart';

part 'quiz_provider.g.dart';

enum GameLevel { easy, medium, hard }
enum QuestionType { whatTimeIsShown, setTheClockTo }

@riverpod
class QuizNotifier extends _$QuizNotifier {
  final _random = Random();
  
  @override
  QuizState build() => QuizState.initial();

  void startQuiz(GameLevel level) {
    state = QuizState(
      level: level,
      currentQuestion: _generateQuestion(level),
      score: 0,
      streak: 0,
      questionsAnswered: 0,
      totalQuestions: level == GameLevel.easy ? 5 : level == GameLevel.medium ? 8 : 10,
      results: [],
    );
  }

  QuizQuestion _generateQuestion(GameLevel level) {
    TimeOfDay time;
    switch (level) {
      case GameLevel.easy:
        // Whole hours only
        time = TimeOfDay(hour: _random.nextInt(12), minute: 0);
        break;
      case GameLevel.medium:
        // 15 min intervals
        final mins = [0, 15, 30, 45];
        time = TimeOfDay(
          hour: _random.nextInt(12),
          minute: mins[_random.nextInt(4)],
        );
        break;
      case GameLevel.hard:
        // Any minute
        time = TimeOfDay(
          hour: _random.nextInt(12),
          minute: _random.nextInt(60),
        );
        break;
    }

    final questionType = _random.nextBool()
        ? QuestionType.whatTimeIsShown
        : QuestionType.setTheClockTo;

    // Generate wrong answers for multiple choice
    final wrongAnswers = _generateWrongAnswers(time, level);

    return QuizQuestion(
      correctTime: time,
      type: questionType,
      choices: level == GameLevel.easy
          ? ([time, ...wrongAnswers]..shuffle(_random))
          : null,
    );
  }

  List<TimeOfDay> _generateWrongAnswers(TimeOfDay correct, GameLevel level) {
    final answers = <TimeOfDay>{};
    while (answers.length < 3) {
      final hour = _random.nextInt(12);
      final minute = level == GameLevel.easy ? 0 : _random.nextInt(60);
      final candidate = TimeOfDay(hour: hour, minute: minute);
      if (candidate != correct) answers.add(candidate);
    }
    return answers.toList();
  }

  void submitAnswer(TimeOfDay answered) {
    final question = state.currentQuestion!;
    final isCorrect = answered.hour == question.correctTime.hour &&
        answered.minute == question.correctTime.minute;

    final pointsEarned = isCorrect
        ? (10 * (state.streak + 1)).clamp(10, 50)
        : 0;

    final newResults = [
      ...state.results,
      QuizResult(
        question: question,
        answered: answered,
        isCorrect: isCorrect,
        pointsEarned: pointsEarned,
      ),
    ];

    final isLastQuestion = state.questionsAnswered + 1 >= state.totalQuestions;

    state = state.copyWith(
      score: state.score + pointsEarned,
      streak: isCorrect ? state.streak + 1 : 0,
      questionsAnswered: state.questionsAnswered + 1,
      results: newResults,
      currentQuestion: isLastQuestion ? null : _generateQuestion(state.level),
      isComplete: isLastQuestion,
      lastAnswerCorrect: isCorrect,
    );
  }
}

class QuizState {
  final GameLevel level;
  final QuizQuestion? currentQuestion;
  final int score;
  final int streak;
  final int questionsAnswered;
  final int totalQuestions;
  final List<QuizResult> results;
  final bool isComplete;
  final bool? lastAnswerCorrect;

  const QuizState({
    required this.level,
    this.currentQuestion,
    required this.score,
    required this.streak,
    required this.questionsAnswered,
    required this.totalQuestions,
    required this.results,
    this.isComplete = false,
    this.lastAnswerCorrect,
  });

  factory QuizState.initial() => const QuizState(
        level: GameLevel.easy,
        score: 0,
        streak: 0,
        questionsAnswered: 0,
        totalQuestions: 5,
        results: [],
      );

  double get accuracy => questionsAnswered == 0
      ? 0
      : results.where((r) => r.isCorrect).length / questionsAnswered;

  QuizState copyWith({
    GameLevel? level,
    QuizQuestion? currentQuestion,
    int? score,
    int? streak,
    int? questionsAnswered,
    int? totalQuestions,
    List<QuizResult>? results,
    bool? isComplete,
    bool? lastAnswerCorrect,
  }) =>
      QuizState(
        level: level ?? this.level,
        currentQuestion: currentQuestion ?? this.currentQuestion,
        score: score ?? this.score,
        streak: streak ?? this.streak,
        questionsAnswered: questionsAnswered ?? this.questionsAnswered,
        totalQuestions: totalQuestions ?? this.totalQuestions,
        results: results ?? this.results,
        isComplete: isComplete ?? this.isComplete,
        lastAnswerCorrect: lastAnswerCorrect ?? this.lastAnswerCorrect,
      );
}
```

---

### `lib/core/l10n/app_en.arb`
```json
{
  "@@locale": "en",
  "appTitle": "TickTock Tales",
  "easyLevel": "Easy",
  "mediumLevel": "Medium",
  "hardLevel": "Hard",
  "whatTimeIsShown": "What time does the clock show?",
  "setClockTo": "Set the clock to {time}",
  "@setClockTo": {
    "placeholders": {
      "time": {"type": "String"}
    }
  },
  "correct": "Great job! 🎉",
  "tryAgain": "Try again! You got this! 💪",
  "score": "Score: {points}",
  "@score": {
    "placeholders": {
      "points": {"type": "int"}
    }
  },
  "streak": "Streak: {count} 🔥",
  "@streak": {
    "placeholders": {
      "count": {"type": "int"}
    }
  },
  "quizComplete": "Quiz Complete!",
  "accuracy": "Accuracy: {percent}%",
  "@accuracy": {
    "placeholders": {
      "percent": {"type": "String"}
    }
  },
  "playAgain": "Play Again",
  "parentMode": "Parent Mode",
  "progressTitle": "Your Child's Progress",
  "resetProgress": "Reset Progress",
  "totalTimeSpent": "Time Spent: {minutes} min",
  "@totalTimeSpent": {
    "placeholders": {
      "minutes": {"type": "int"}
    }
  },
  "badges": "Badges",
  "settings": "Settings",
  "language": "Language"
}
```

### `lib/core/l10n/app_ar.arb`
```json
{
  "@@locale": "ar",
  "appTitle": "حكايات الساعة",
  "easyLevel": "سهل",
  "mediumLevel": "متوسط",
  "hardLevel": "صعب",
  "whatTimeIsShown": "ما الوقت الذي تُظهره الساعة؟",
  "setClockTo": "اضبط الساعة على {time}",
  "correct": "أحسنت! 🎉",
  "tryAgain": "حاول مرة أخرى! 💪",
  "score": "النقاط: {points}",
  "streak": "سلسلة: {count} 🔥",
  "quizComplete": "اكتمل الاختبار!",
  "accuracy": "الدقة: {percent}%",
  "playAgain": "العب مرة أخرى",
  "parentMode": "وضع الوالدين",
  "progressTitle": "تقدم طفلك",
  "resetProgress": "إعادة تعيين التقدم",
  "totalTimeSpent": "الوقت المستغرق: {minutes} دقيقة",
  "badges": "الشارات",
  "settings": "الإعدادات",
  "language": "اللغة"
}
```

---

### Gamification — Badges
```dart
// lib/features/gamification/domain/badge.dart
enum BadgeType {
  firstCorrect,    // First right answer
  streak5,         // 5 in a row
  streak10,        // 10 in a row
  perfectEasy,     // 100% on easy
  perfectMedium,   // 100% on medium
  perfectHard,     // 100% on hard
  speedster,       // Hard level under time limit
  allLevels,       // Completed all 3 levels
}

class AppBadge {
  final BadgeType type;
  final String nameKey;      // l10n key
  final String description;
  final String iconAsset;
  final bool isUnlocked;
  final DateTime? unlockedAt;

  const AppBadge({
    required this.type,
    required this.nameKey,
    required this.description,
    required this.iconAsset,
    this.isUnlocked = false,
    this.unlockedAt,
  });
}
```

---

## 🧪 Testing

```bash
# Unit tests
flutter test test/unit/

# Widget tests
flutter test test/widget/

# Integration tests
flutter test integration_test/
```

Key test files:
- `test/unit/time_utils_test.dart`
- `test/unit/question_generator_test.dart`
- `test/widget/analog_clock_test.dart`
- `test/widget/quiz_screen_test.dart`

---

## 🚀 v2 Improvements

1. **AI Voice Reading** — Use `flutter_tts` to read the time aloud, supporting all 4 languages
2. **Adaptive Difficulty** — Track accuracy per 5 questions; auto-adjust level
3. **Avatar Customization** — Unlock hats, clothes, accessories via points
4. **Local Leaderboard** — SQLite with `sqflite`, show top 5 sessions
5. **Offline-first sync prep** — Abstract repository layer already supports future backend
6. **Accessibility** — Full `Semantics` wrapping for screen readers
7. **Parent PIN lock** — Protect parent mode with a 4-digit PIN

---

## Build for Production

```bash
# Android APK
flutter build apk --release

# Android App Bundle (Play Store)
flutter build appbundle --release

# iOS (requires Mac + Xcode)
flutter build ios --release
```
