import 'dart:math';
import 'package:flutter/material.dart';
import 'package:flutter_tts/flutter_tts.dart';
import 'package:speech_to_text/speech_to_text.dart' as stt;

void main() {
  runApp(const RehabDemoApp());
}

class RehabDemoApp extends StatelessWidget {
  const RehabDemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: '청능재활 데모',
      theme: ThemeData(colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo), useMaterial3: true),
      home: const DemoHome(),
    );
  }
}

class DemoHome extends StatefulWidget {
  const DemoHome({super.key});

  @override
  State<DemoHome> createState() => _DemoHomeState();
}

class _DemoHomeState extends State<DemoHome> {
  final FlutterTts tts = FlutterTts();
  final stt.SpeechToText speech = stt.SpeechToText();

  final List<String> prompts = const [
    '가',
    '바다',
    '서랍 안에 가위가 있어요.',
    '산에 가서 사진을 찍었다.',
  ];

  String target = '서랍 안에 가위가 있어요.';
  bool showAnswer = false;

  bool listening = false;
  String recognized = '';
  double score = 0;
  String feedback = '';

  @override
  void initState() {
    super.initState();
    _initTts();
  }

  Future<void> _initTts() async {
    await tts.setLanguage('ko-KR');
    await tts.setSpeechRate(0.45);
    await tts.setPitch(1.0);
  }

  @override
  void dispose() {
    tts.stop();
    speech.stop();
    super.dispose();
  }

  Future<void> speak() async {
    await tts.stop();
    await tts.speak(target);
  }

  Future<void> startListen() async {
    await tts.stop();

    final ok = await speech.initialize(
      onError: (e) => _toast('STT 오류: ${e.errorMsg}'),
      onStatus: (s) {
        if (s == 'notListening') setState(() => listening = false);
      },
    );
    if (!ok) {
      _toast('이 기기에서 STT를 사용할 수 없습니다.');
      return;
    }

    setState(() {
      listening = true;
      recognized = '';
      score = 0;
      feedback = '';
    });

    await speech.listen(
      localeId: 'ko_KR',
      listenFor: const Duration(seconds: 8),
      pauseFor: const Duration(seconds: 2),
      onResult: (r) => setState(() => recognized = r.recognizedWords),
    );
  }

  Future<void> stopAndScore() async {
    await speech.stop();
    setState(() => listening = false);

    final a = _normalize(target);
    final b = _normalize(recognized);

    if (b.isEmpty) {
      setState(() {
        score = 0;
        feedback = '인식된 음성이 없습니다. 다시 말해보세요.';
      });
      return;
    }

    final sim = _similarity(a, b);
    final sc = (sim * 100).clamp(0, 100);

    setState(() {
      score = sc;
      feedback = _simpleFeedback(target, recognized);
    });
  }

  void nextPrompt() {
    final r = Random().nextInt(prompts.length);
    setState(() {
      target = prompts[r];
      recognized = '';
      score = 0;
      feedback = '';
      showAnswer = false;
      listening = false;
    });
  }

  void _toast(String msg) => ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(msg)));

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('청능재활 데모 (TTS→STT→점수)')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          Card(
            child: Padding(
              padding: const EdgeInsets.all(14),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text('문항', style: TextStyle(fontWeight: FontWeight.w900)),
                  const SizedBox(height: 8),
                  Text(showAnswer ? target : '정답 숨김 (버튼으로 공개 가능)'),
                  const SizedBox(height: 8),
                  SwitchListTile(
                    contentPadding: EdgeInsets.zero,
                    title: const Text('정답 보기'),
                    value: showAnswer,
                    onChanged: (v) => setState(() => showAnswer = v),
                  ),
                  const SizedBox(height: 8),
                  Row(
                    children: [
                      Expanded(
                        child: FilledButton.icon(
                          onPressed: speak,
                          icon: const Icon(Icons.volume_up),
                          label: const Text('불러주기(TTS)'),
                        ),
                      ),
                      const SizedBox(width: 10),
                      Expanded(
                        child: FilledButton.icon(
                          onPressed: listening ? stopAndScore : startListen,
                          icon: Icon(listening ? Icons.stop : Icons.mic),
                          label: Text(listening ? '정지/채점' : '따라 말하기'),
                        ),
                      ),
                    ],
                  ),
                  const SizedBox(height: 10),
                  OutlinedButton.icon(
                    onPressed: nextPrompt,
                    icon: const Icon(Icons.skip_next),
                    label: const Text('다음 문항'),
                  ),
                ],
              ),
            ),
          ),
          const SizedBox(height: 12),
          Card(
            elevation: 0,
            color: Theme.of(context).colorScheme.surfaceContainerHighest,
            child: Padding(
              padding: const EdgeInsets.all(14),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text('인식 결과', style: TextStyle(fontWeight: FontWeight.w900)),
                  const SizedBox(height: 8),
                  Text(recognized.isEmpty ? '(아직 없음)' : recognized),
                  const SizedBox(height: 12),
                  const Text('점수', style: TextStyle(fontWeight: FontWeight.w900)),
                  const SizedBox(height: 8),
                  ClipRRect(
                    borderRadius: BorderRadius.circular(999),
                    child: LinearProgressIndicator(value: (score / 100).clamp(0, 1.0), minHeight: 10),
                  ),
                  const SizedBox(height: 8),
                  Text('${score.toStringAsFixed(1)} / 100'),
                  const SizedBox(height: 12),
                  const Text('피드백', style: TextStyle(fontWeight: FontWeight.w900)),
                  const SizedBox(height: 8),
                  Text(feedback.isEmpty ? '(채점 후 표시)' : feedback),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }

  String _normalize(String s) {
    final trimmed = s.trim();
    final removedPunct = trimmed.replaceAll(RegExp(r'[^\uAC00-\uD7A3a-zA-Z0-9 ]'), '');
    return removedPunct.replaceAll(' ', '');
  }

  double _similarity(String a, String b) {
    if (a.isEmpty && b.isEmpty) return 1.0;
    if (a.isEmpty || b.isEmpty) return 0.0;
    final dist = _levenshtein(a, b);
    final maxLen = max(a.length, b.length);
    return 1.0 - (dist / maxLen);
  }

  int _levenshtein(String s, String t) {
    final m = s.length, n = t.length;
    final dp = List.generate(m + 1, (_) => List<int>.filled(n + 1, 0));
    for (var i = 0; i <= m; i++) dp[i][0] = i;
    for (var j = 0; j <= n; j++) dp[0][j] = j;

    for (var i = 1; i <= m; i++) {
      for (var j = 1; j <= n; j++) {
        final cost = s[i - 1] == t[j - 1] ? 0 : 1;
        dp[i][j] = min(
          min(dp[i - 1][j] + 1, dp[i][j - 1] + 1),
          dp[i - 1][j - 1] + cost,
        );
      }
    }
    return dp[m][n];
  }

  String _simpleFeedback(String target, String recognized) {
    final tWords = target.trim().split(RegExp(r'\s+')).where((w) => w.isNotEmpty).toList();
    final rWords = recognized.trim().split(RegExp(r'\s+')).where((w) => w.isNotEmpty).toList();
    if (rWords.isEmpty) return '인식된 단어가 없습니다.';

    final missing = tWords.where((w) => !rWords.contains(w)).toList();
    final extra = rWords.where((w) => !tWords.contains(w)).toList();

    final buf = StringBuffer();
    if (missing.isNotEmpty) buf.writeln('누락 추정: ${missing.join(', ')}');
    if (extra.isNotEmpty) buf.writeln('추가/오인식 추정: ${extra.join(', ')}');
    if (buf.isEmpty) buf.writeln('좋습니다. 거의 동일하게 말했어요.');
    return buf.toString().trim();
  }
}
