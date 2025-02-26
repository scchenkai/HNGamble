彩票投注程序开发实施计划
项目概述
项目名称：彩票投注管理系统（LotteryBet Pro）
项目目标：开发一款支持投注站管理、玩法管理、用户投注管理、报表统计及风险控制的彩票投注程序，具备 Web 端和手机端兼容性（初期快速开发 Web 端）。
目标用户：彩票投注站运营者、管理员。
当前日期：2025年2月24日
更新需求分析
投注站管理：
支持多个投注站独立设置返点和赔率。
投注时无需用户 ID，每日每个投注站从 1 开始递增生成投注编号。
玩法管理：
支持多种玩法，具备扩展性。
用户投注管理：
记录当天投注编号、金额、玩法、投注站数据。
高风险赔付提醒。
根据开奖结果和赔率表计算中奖金额及返点。
支持接收用户不完整聊天记录，转大模型处理，管理员确认后自动投注。
报表统计导出：
提供当日、当月投注数据统计，异常数据提醒，支持导出。
风险控制与订单分配：
根据投注情况，智能分配订单，降低风险。
技术栈
前端：React.js + Ant Design Mobile（支持 Web 和手机端响应式设计，快速开发 Web 端）。
后端：Python + Flask（轻量级框架，适合快速开发）。
数据库：
本地开发：SQLite（轻量、无需服务器，适合开发）。
生产环境：PostgreSQL（高性能，支持高并发）。
缓存：Redis（缓存赔率和每日编号）。
大模型接口：对接 Grok 3 API（或类似模型）处理聊天记录。
部署：Docker + AWS EC2。
数据库表设计及建表 SQL
1. stations（投注站表）
SQLite：
sql
自动换行
复制
CREATE TABLE stations (
    station_id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    address TEXT,
    rebate_rate REAL DEFAULT 0.00,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
PostgreSQL：
sql
自动换行
复制
CREATE TABLE stations (
    station_id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    address VARCHAR(255),
    rebate_rate NUMERIC(5,2) DEFAULT 0.00,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
2. play_types（玩法表）
SQLite：
sql
自动换行
复制
CREATE TABLE play_types (
    play_id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    description TEXT,
    default_odds REAL DEFAULT 1.00
);
PostgreSQL：
sql
自动换行
复制
CREATE TABLE play_types (
    play_id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    description TEXT,
    default_odds NUMERIC(10,2) DEFAULT 1.00
);
3. station_odds（投注站赔率表）
SQLite：
sql
自动换行
复制
CREATE TABLE station_odds (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    station_id INTEGER NOT NULL,
    play_id INTEGER NOT NULL,
    odds REAL DEFAULT 1.00,
    FOREIGN KEY (station_id) REFERENCES stations(station_id),
    FOREIGN KEY (play_id) REFERENCES play_types(play_id)
);
PostgreSQL：
sql
自动换行
复制
CREATE TABLE station_odds (
    id SERIAL PRIMARY KEY,
    station_id INTEGER NOT NULL,
    play_id INTEGER NOT NULL,
    odds NUMERIC(10,2) DEFAULT 1.00,
    FOREIGN KEY (station_id) REFERENCES stations(station_id),
    FOREIGN KEY (play_id) REFERENCES play_types(play_id)
);
4. bets（投注表）
SQLite：
sql
自动换行
复制
CREATE TABLE bets (
    bet_id INTEGER PRIMARY KEY,
    station_id INTEGER NOT NULL,
    bet_number INTEGER NOT NULL,
    play_id INTEGER NOT NULL,
    amount REAL NOT NULL,
    bet_content TEXT NOT NULL,
    status TEXT DEFAULT 'pending' CHECK(status IN ('pending', 'won', 'lost')),
    result_amount REAL DEFAULT 0.00,
    bet_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (station_id) REFERENCES stations(station_id),
    FOREIGN KEY (play_id) REFERENCES play_types(play_id)
);
PostgreSQL：
sql
自动换行
复制
CREATE TABLE bets (
    bet_id BIGINT PRIMARY KEY,
    station_id INTEGER NOT NULL,
    bet_number INTEGER NOT NULL,
    play_id INTEGER NOT NULL,
    amount NUMERIC(10,2) NOT NULL,
    bet_content VARCHAR(255) NOT NULL,
    status VARCHAR(7) DEFAULT 'pending' CHECK(status IN ('pending', 'won', 'lost')),
    result_amount NUMERIC(10,2) DEFAULT 0.00,
    bet_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (station_id) REFERENCES stations(station_id),
    FOREIGN KEY (play_id) REFERENCES play_types(play_id)
);
5. results（开奖结果表）
SQLite：
sql
自动换行
复制
CREATE TABLE results (
    result_id INTEGER PRIMARY KEY AUTOINCREMENT,
    date DATE NOT NULL,
    play_id INTEGER NOT NULL,
    result TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (play_id) REFERENCES play_types(play_id)
);
PostgreSQL：
sql
自动换行
复制
CREATE TABLE results (
    result_id SERIAL PRIMARY KEY,
    date DATE NOT NULL,
    play_id INTEGER NOT NULL,
    result VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (play_id) REFERENCES play_types(play_id)
);
6. chat_inputs（聊天记录输入表）
SQLite：
sql
自动换行
复制
CREATE TABLE chat_inputs (
    input_id INTEGER PRIMARY KEY AUTOINCREMENT,
    station_id INTEGER NOT NULL,
    raw_input TEXT NOT NULL,
    processed_input TEXT,
    status TEXT DEFAULT 'pending' CHECK(status IN ('pending', 'approved', 'rejected')),
    bet_id INTEGER,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (station_id) REFERENCES stations(station_id),
    FOREIGN KEY (bet_id) REFERENCES bets(bet_id)
);
PostgreSQL：
sql
自动换行
复制
CREATE TABLE chat_inputs (
    input_id SERIAL PRIMARY KEY,
    station_id INTEGER NOT NULL,
    raw_input TEXT NOT NULL,
    processed_input TEXT,
    status VARCHAR(8) DEFAULT 'pending' CHECK(status IN ('pending', 'approved', 'rejected')),
    bet_id BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (station_id) REFERENCES stations(station_id),
    FOREIGN KEY (bet_id) REFERENCES bets(bet_id)
);
开发步骤与代码实施计划
1. 项目初始化（1周）
任务：
初始化项目结构，配置 Flask，支持 SQLite（开发）和 PostgreSQL（生产）。
搭建基本路由（/stations, /bets, /chat 等）。
代码示例（后端 - Flask 初始化）：
python
自动换行
复制
# app.py
from flask import Flask, request, jsonify
import sqlite3
import psycopg2
from os import environ

app = Flask(__name__)
ENV = environ.get('ENV', 'development')  # 默认开发环境

def get_db_connection():
    if ENV == 'development':
        conn = sqlite3.connect('lottery_bet.db')
        conn.row_factory = sqlite3.Row
        return conn
    else:
        conn = psycopg2.connect(
            dbname="lottery_bet",
            user="postgres",
            password="password",
            host="localhost",
            port="5432"
        )
        return conn

@app.route('/')
def hello():
    return jsonify({"message": "Welcome to LotteryBet Pro"})

if __name__ == '__main__':
    app.run(debug=True, port=5000)
相关文件：
requirements.txt：添加依赖 flask, psycopg2-binary, requests, python-dotenv。
.env：存储生产环境 PostgreSQL 凭证。
前端初始化：
任务：使用 create-react-app 初始化 React 项目，安装 Ant Design Mobile。
代码示例：
bash
自动换行
复制
npx create-react-app lottery-bet-frontend
cd lottery-bet-frontend
npm install antd-mobile axios
2. 数据库创建与投注站管理（2周）
任务：
本地运行 SQLite 建表脚本，生产环境部署 PostgreSQL。
实现投注站 CRUD 接口。
代码示例（创建投注站 - Python）：
python
自动换行
复制
# routes/stations.py
@app.route('/stations', methods=['POST'])
def create_station():
    data = request.get_json()
    name, address, rebate_rate = data['name'], data['address'], data['rebate_rate']
    
    conn = get_db_connection()
    cursor = conn.cursor()
    if ENV == 'development':
        cursor.execute(
            "INSERT INTO stations (name, address, rebate_rate) VALUES (?, ?, ?)",
            (name, address, rebate_rate)
        )
    else:
        cursor.execute(
            "INSERT INTO stations (name, address, rebate_rate) VALUES (%s, %s, %s)",
            (name, address, rebate_rate)
        )
    conn.commit()
    station_id = cursor.lastrowid if ENV == 'development' else cursor.lastval()
    cursor.close()
    conn.close()
    return jsonify({"id": station_id, "message": "投注站创建成功"}), 201
前端示例（React 表单）：
jsx
自动换行
复制
// src/components/StationForm.jsx
import { Form, Input, Button } from 'antd-mobile';
import axios from 'axios';

const StationForm = () => {
  const onFinish = (values) => {
    axios.post('http://localhost:5000/stations', values)
      .then(res => alert(res.data.message))
      .catch(err => console.error(err));
  };

  return (
    <Form onFinish={onFinish}>
      <Form.Item name="name" label="名称"><Input placeholder="请输入名称" /></Form.Item>
      <Form.Item name="address" label="地址"><Input placeholder="请输入地址" /></Form.Item>
      <Form.Item name="rebate_rate" label="返点"><Input type="number" placeholder="如 5.00" /></Form.Item>
      <Button type="primary" htmlType="submit">提交</Button>
    </Form>
  );
};
export default StationForm;
3. 玩法管理与赔率设置（2周）
任务：
实现玩法添加和赔率自定义。
代码示例（查询投注站赔率 - Python）：
python
自动换行
复制
# routes/station_odds.py
@app.route('/station_odds/<int:station_id>', methods=['GET'])
def get_station_odds(station_id):
    conn = get_db_connection()
    cursor = conn.cursor()
    query = "SELECT p.name, so.odds FROM station_odds so JOIN play_types p ON so.play_id = p.play_id WHERE station_id = ?"
    param = (station_id,) if ENV == 'development' else [station_id]
    cursor.execute(query if ENV == 'development' else query.replace('?', '%s'), param)
    odds = [dict(row) for row in cursor.fetchall()]
    cursor.close()
    conn.close()
    return jsonify(odds)
4. 用户投注管理与聊天记录处理（3周）
任务：
实现投注记录（每日编号递增）。
添加聊天记录接口，调用大模型处理。
代码示例（生成投注记录 - Python）：
python
自动换行
复制
# routes/bets.py
from datetime import datetime

@app.route('/bets', methods=['POST'])
def create_bet():
    data = request.get_json()
    station_id, play_id, amount, bet_content = data['station_id'], data['play_id'], data['amount'], data['bet_content']
    date_str = datetime.now().strftime('%Y%m%d')
    
    conn = get_db_connection()
    cursor = conn.cursor()
    query = "SELECT bet_number FROM bets WHERE station_id = ? AND bet_time LIKE ? ORDER BY bet_number DESC LIMIT 1"
    params = (station_id, f"{date_str}%") if ENV == 'development' else [station_id, f"{date_str}%"]
    cursor.execute(query if ENV == 'development' else query.replace('?', '%s'), params)
    last_number = cursor.fetchone()
    bet_number = (last_number[0] + 1) if last_number else 1
    bet_id = int(f"{date_str}{station_id}{bet_number:04d}")

    insert_query = "INSERT INTO bets (bet_id, station_id, bet_number, play_id, amount, bet_content, status) VALUES (?, ?, ?, ?, ?, ?, 'pending')"
    insert_params = (bet_id, station_id, bet_number, play_id, amount, bet_content) if ENV == 'development' else [bet_id, station_id, bet_number, play_id, amount, bet_content]
    cursor.execute(insert_query if ENV == 'development' else insert_query.replace('?', '%s'), insert_params)
    conn.commit()
    cursor.close()
    conn.close()
    return jsonify({"bet_id": bet_id, "message": "投注成功"}), 201
代码示例（聊天记录处理 - Python）：
python
自动换行
复制
# routes/chat.py
import requests

@app.route('/chat', methods=['POST'])
def process_chat():
    data = request.get_json()
    station_id, raw_input = data['station_id'], data['raw_input']
    
    # 调用Grok 3 API（假设端点）
    grok_response = requests.post('https://api.x.ai/grok', json={"prompt": f"将以下投注请求转为标准格式：{raw_input}"})
    processed_input = grok_response.json().get('result')

    conn = get_db_connection()
    cursor = conn.cursor()
    query = "INSERT INTO chat_inputs (station_id, raw_input, processed_input, status) VALUES (?, ?, ?, 'pending')"
    params = (station_id, raw_input, processed_input) if ENV == 'development' else [station_id, raw_input, processed_input]
    cursor.execute(query if ENV == 'development' else query.replace('?', '%s'), params)
    conn.commit()
    input_id = cursor.lastrowid
    cursor.close()
    conn.close()
    return jsonify({"input_id": input_id, "processed_input": processed_input}), 201

@app.route('/chat/approve/<int:input_id>', methods=['POST'])
def approve_chat(input_id):
    conn = get_db_connection()
    cursor = conn.cursor()
    query = "SELECT * FROM chat_inputs WHERE input_id = ?"
    cursor.execute(query if ENV == 'development' else query.replace('?', '%s'), (input_id,) if ENV == 'development' else [input_id])
    chat = cursor.fetchone()
    if dict(chat)['status'] != 'pending':
        return jsonify({"error": "已处理"}), 400

    play, number, amount = dict(chat)['processed_input'].split('，')
    play_id = get_play_id(play.split('：')[1])  # 自定义函数获取玩法ID
    bet_data = {
        "station_id": dict(chat)['station_id'],
        "play_id": play_id,
        "amount": float(amount.split('：')[1]),
        "bet_content": number
    }
    bet_response = requests.post('http://localhost:5000/bets', json=bet_data).json()

    update_query = "UPDATE chat_inputs SET status = 'approved', bet_id = ? WHERE input_id = ?"
    update_params = (bet_response['bet_id'], input_id) if ENV == 'development' else [bet_response['bet_id'], input_id]
    cursor.execute(update_query if ENV == 'development' else update_query.replace('?', '%s'), update_params)
    conn.commit()
    cursor.close()
    conn.close()
    return jsonify({"message": "投注已完成"})
5. 报表统计与风险控制（3周）
任务：
实现数据统计和导出。
添加高风险提醒逻辑。
代码示例（每日统计 - Python）：
python
自动换行
复制
# routes/reports.py
@app.route('/reports/daily/<date>', methods=['GET'])
def daily_report(date):
    conn = get_db_connection()
    cursor = conn.cursor()
    query = "SELECT station_id, SUM(amount) as total_bet, SUM(result_amount) as total_payout FROM bets WHERE bet_time LIKE ? GROUP BY station_id"
    params = (f"{date}%",) if ENV == 'development' else [f"{date}%"]
    cursor.execute(query if ENV == 'development' else query.replace('?', '%s'), params)
    report = [dict(row) for row in cursor.fetchall()]
    cursor.close()
    conn.close()
    return jsonify(report)
6. 测试与部署（2周）
任务：
单元测试（使用 pytest 测试 API）。
部署到 AWS，使用 Docker 和 Nginx。
代码示例（测试用例）：
python
自动换行
复制
# tests/test_stations.py
import pytest
from app import app

def test_create_station():
    client = app.test_client()
    response = client.post('/stations', json={"name": "站A", "address": "北京", "rebate_rate": 5.00})
    assert response.status_code == 201
    assert response.json['message'] == "投注站创建成功"
注意事项
新手提示：
本地开发：安装 Python 3.9+，无需额外安装 SQLite（内置支持），运行 pip install -r requirements.txt。
生产环境：安装 PostgreSQL，导入建表 SQL（psql -U postgres -d lottery_bet -f schema.sql）。
前端：运行 npm start 启动 React 项目。
依赖管理：
后端：pip install flask psycopg2-binary requests python-dotenv。
前端：npm install antd-mobile axios。
数据库切换：
本地使用 lottery_bet.db 文件，生产环境配置 .env 文件中的 PostgreSQL 连接参数。
