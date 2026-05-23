import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'package:fl_chart/fl_chart.dart';
import 'package:intl/intl.dart'; // 日付フォーマット用に追加が必要な場合があります

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'EA Dashboard',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true, // モダンなUIのために有効化
      ),
      home: const DashboardScreen(),
    );
  }
}

// データモデルクラス
class TradeRecord {
  final String date;
  final String eaName;
  final String rawProfit;
  final String rawMaxDD;
  final double profitValue;

  TradeRecord({
    required this.date,
    required this.eaName,
    required this.rawProfit,
    required this.rawMaxDD,
    required this.profitValue,
  });

  factory TradeRecord.fromJson(Map<String, dynamic> json) {
    // 利益のパース: '推奨超え'などの文字列なら0にする
    double parsedProfit = 0.0;
    String rawProfitStr = json['収支(円/結果)']?.toString() ?? '0';
    if (double.tryParse(rawProfitStr) != null) {
      parsedProfit = double.parse(rawProfitStr);
    }

    return TradeRecord(
      date: json['日付']?.toString() ?? '',
      eaName: json['EA名']?.toString() ?? '',
      rawProfit: rawProfitStr,
      rawMaxDD: json['最大DD(%)']?.toString() ?? '-',
      profitValue: parsedProfit,
    );
  }
}

class DashboardScreen extends StatefulWidget {
  const DashboardScreen({Key? key}) : super(key: key);

  @override
  _DashboardScreenState createState() => _DashboardScreenState();
}

class _DashboardScreenState extends State<DashboardScreen> {
  // === 【ここにご自身のGAS URLを入力】 ===
  final String _apiUrl =
      'https://script.google.com/macros/s/AKfycbwKF3noJDbCFVl1LybfHbyjEsXWGVApxkqini26fb4RimlpgGbRPJqIyPCkMK6D-7RD/exec';

  List<TradeRecord> _records = [];
  bool _isLoading = true;
  String _errorMessage = '';

  @override
  void initState() {
    super.initState();
    _fetchData();
  }

  // 1. httpパッケージを使ってデータを取得するメソッド
  Future<void> _fetchData() async {
    try {
      final response = await http.get(Uri.parse(_apiUrl));

      if (response.statusCode == 200) {
        // GASから返ってくるJSONはUTF-8でデコード
        List<dynamic> jsonData = json.decode(utf8.decode(response.bodyBytes));
        setState(() {
          _records = jsonData.map((item) => TradeRecord.fromJson(item)).toList();
          _isLoading = false;
        });
      } else {
        setState(() {
          _errorMessage = 'データの取得に失敗しました (Status: ${response.statusCode})';
          _isLoading = false;
        });
      }
    } catch (e) {
      setState(() {
        _errorMessage = 'エラーが発生しました: $e';
        _isLoading = false;
      });
    }
  }

  // 合計利益を計算する
  double get _totalProfit {
    return _records.fold(0, (sum, item) => sum + item.profitValue);
  }

  // fl_chart用のデータポイントを作成する
  List<FlSpot> _generateChartSpots() {
    List<FlSpot> spots = [];
    for (int i = 0; i < _records.length; i++) {
      spots.add(FlSpot(i.toDouble(), _records[i].profitValue));
    }
    return spots;
  }

  @override
  Widget build(BuildContext context) {
    // 5. ローディング中の表示
    if (_isLoading) {
      return const Scaffold(
        body: Center(child: CircularProgressIndicator()),
      );
    }

    // エラー時の表示
    if (_errorMessage.isNotEmpty) {
      return Scaffold(
        appBar: AppBar(title: const Text('Dashboard')),
        body: Center(child: Text(_errorMessage, style: const TextStyle(color: Colors.red))),
      );
    }

    return Scaffold(
      backgroundColor: Colors.grey[100],
      appBar: AppBar(
        title: const Text('EA 収支ダッシュボード'),
        backgroundColor: Colors.white,
        elevation: 0,
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () {
              setState(() {
                _isLoading = true;
                _errorMessage = '';
              });
              _fetchData();
            },
          )
        ],
      ),
      body: SafeArea(
        child: Column(
          children: [
            // 2. 画面上部の「合計利益」表示
            _buildSummaryCard(),
            
            // 3. fl_chartを使った折れ線グラフ
            Expanded(
              flex: 2,
              child: _buildChartSection(),
            ),
            
            const Padding(
              padding: EdgeInsets.symmetric(horizontal: 16.0, vertical: 8.0),
              child: Align(
                alignment: Alignment.centerLeft,
                child: Text('日々の詳細データ', style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16)),
              ),
            ),
            
            // 4. データ一覧のListView
            Expanded(
              flex: 3,
              child: _buildListView(),
            ),
          ],
        ),
      ),
    );
  }

  // 合計利益カードのUI
  Widget _buildSummaryCard() {
    // カンマ区切りのフォーマット
    final formatter = NumberFormat('#,##0');
    final formattedTotal = formatter.format(_totalProfit);

    return Container(
      margin: const EdgeInsets.all(16.0),
      padding: const EdgeInsets.all(24.0),
      decoration: BoxDecoration(
        color: Colors.blueAccent,
        borderRadius: BorderRadius.circular(16),
        boxShadow: [
          BoxShadow(
            color: Colors.blue.withOpacity(0.3),
            blurRadius: 10,
            offset: const Offset(0, 5),
          ),
        ],
      ),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Icon(Icons.account_balance_wallet, color: Colors.white, size: 40),
          const SizedBox(width: 16),
          Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text('累計合計利益', style: TextStyle(color: Colors.white70, fontSize: 14)),
              Text(
                '¥$formattedTotal',
                style: const TextStyle(color: Colors.white, fontSize: 32, fontWeight: FontWeight.bold),
              ),
            ],
          ),
        ],
      ),
    );
  }

  // グラフセクションのUI
  Widget _buildChartSection() {
    if (_records.isEmpty) {
      return const Center(child: Text('データがありません'));
    }

    return Container(
      margin: const EdgeInsets.symmetric(horizontal: 16.0),
      padding: const EdgeInsets.all(16.0),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(16),
        boxShadow: [
          BoxShadow(color: Colors.grey.withOpacity(0.1), blurRadius: 10, offset: const Offset(0, 5)),
        ],
      ),
      child: LineChart(
        LineChartData(
          gridData: const FlGridData(show: true, drawVerticalLine: false),
          titlesData: FlTitlesData(
            show: true,
            rightTitles: const AxisTitles(sideTitles: SideTitles(showTitles: false)),
            topTitles: const AxisTitles(sideTitles: SideTitles(showTitles: false)),
            // 下部のX軸（日付）の設定
            bottomTitles: AxisTitles(
              sideTitles: SideTitles(
                showTitles: true,
                reservedSize: 30,
                interval: (_records.length / 5).ceilToDouble(), // ラベルが重ならないように間引く
                getTitlesWidget: (value, meta) {
                  int index = value.toInt();
                  if (index >= 0 && index < _records.length) {
                    // 日付文字列（例: 2026/05/20）から月日だけ（05/20）を抽出
                    String dateStr = _records[index].date;
                    if (dateStr.length >= 10) {
                      return Padding(
                        padding: const EdgeInsets.only(top: 8.0),
                        child: Text(dateStr.substring(5, 10), style: const TextStyle(fontSize: 10)),
                      );
                    }
                  }
                  return const Text('');
                },
              ),
            ),
          ),
          borderData: FlBorderData(show: false),
          // グラフの線の設定
          lineBarsData: [
            LineChartBarData(
              spots: _generateChartSpots(),
              isCurved: true, // 曲線を滑らかにする
              color: Colors.blue,
              barWidth: 3,
              isStrokeCapRound: true,
              dotData: const FlDotData(show: false), // ドットを非表示
              belowBarData: BarAreaData(
                show: true,
                color: Colors.blue.withOpacity(0.1), // グラフの下を薄く塗りつぶす
              ),
            ),
          ],
        ),
      ),
    );
  }

  // データリストのUI
  Widget _buildListView() {
    // 最新のデータが上に来るように逆順で表示する
    final reversedRecords = _records.reversed.toList();

    return ListView.builder(
      padding: const EdgeInsets.symmetric(horizontal: 16.0),
      itemCount: reversedRecords.length,
      itemBuilder: (context, index) {
        final record = reversedRecords[index];
        // 利益が0以上なら青、0ならグレーのテキストにする
        final isProfit = record.profitValue > 0;

        return Card(
          elevation: 0,
          margin: const EdgeInsets.only(bottom: 8.0),
          color: Colors.white,
          shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
          child: ListTile(
            leading: CircleAvatar(
              backgroundColor: isProfit ? Colors.blue.withOpacity(0.1) : Colors.grey.withOpacity(0.1),
              child: Icon(Icons.show_chart, color: isProfit ? Colors.blue : Colors.grey),
            ),
            title: Text(record.eaName, style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 14)),
            subtitle: Text('${record.date}  |  DD: ${record.rawMaxDD}%', style: const TextStyle(fontSize: 12)),
            trailing: Text(
              // "推奨超え" などの生データをそのまま表示する
              record.rawProfit == '推奨超え' ? record.rawProfit : '¥${record.rawProfit}',
              style: TextStyle(
                fontWeight: FontWeight.bold,
                fontSize: 16,
                color: isProfit || record.rawProfit == '推奨超え' ? Colors.blue : Colors.grey,
              ),
            ),
          ),
        );
      },
    );
  }
}
