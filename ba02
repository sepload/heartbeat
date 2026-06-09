import streamlit as st
import pandas as pd
import plotly.express as px
from heartbeat_simulator import generate_heartbeat_data

st.set_page_config(page_title="无人机心跳监测", layout="wide")
st.title("🚁 无人机通信“心跳”监测可视化")
st.markdown("模拟无人机每3秒发送一次心跳包，自动检测掉线并展示")

# 侧边栏配置
st.sidebar.header("模拟参数")
duration = st.sidebar.slider("模拟时长 (分钟)", 5, 60, 30, step=5)
normal_interval = st.sidebar.number_input("正常心跳间隔 (秒)", min_value=1, max_value=10, value=3)
offline_threshold = st.sidebar.number_input("掉线阈值 (秒)", min_value=2, max_value=10, value=5)

if st.sidebar.button("重新生成数据"):
    st.session_state['data'] = generate_heartbeat_data(
        duration_minutes=duration,
        normal_interval=normal_interval,
        offline_threshold=offline_threshold
    )

# 初始化或加载数据
if 'data' not in st.session_state:
    st.session_state['data'] = generate_heartbeat_data(duration, normal_interval, offline_threshold)

df = st.session_state['data'].copy()

# 主区域展示
col1, col2 = st.columns(2)
with col1:
    st.metric("总心跳包数", len(df))
with col2:
    offline_count = df['掉线标记'].sum()
    st.metric("掉线事件次数", offline_count)

# 心跳间隔折线图
st.subheader("📈 心跳间隔变化（秒）")
fig_interval = px.line(
    df, x='序号', y='间隔',
    title="心跳间隔时序图 (超过阈值红色高亮)",
    labels={'序号': '心跳序号', '间隔': '间隔(秒)'}
)
# 高亮掉线点（间隔>阈值）
offline_points = df[df['掉线标记']]
fig_interval.add_scatter(
    x=offline_points['序号'], y=offline_points['间隔'],
    mode='markers', marker=dict(color='red', size=8),
    name='掉线恢复点'
)
fig_interval.add_hline(y=offline_threshold, line_dash="dash", line_color="orange", annotation_text="掉线阈值")
st.plotly_chart(fig_interval, use_container_width=True)

# 心跳时间戳分布（作为散点图查看时间序列）
st.subheader("⏱️ 心跳时间序列")
df['时间戳_str'] = df['时间戳'].dt.strftime('%H:%M:%S')
fig_time = px.scatter(
    df, x='序号', y='时间戳_str',
    title="心跳发送时刻",
    labels={'序号': '心跳序号', '时间戳_str': '发送时间'},
    hover_data=['间隔']
)
st.plotly_chart(fig_time, use_container_width=True)

# 数据表格（可折叠）
with st.expander("查看原始心跳数据"):
    st.dataframe(df[['序号', '时间戳', '间隔', '掉线标记']])

st.markdown("---")
st.caption("模拟数据生成逻辑：每3秒一个心跳，随机模拟10~20秒的掉线，间隔超过5秒即判定为掉线")
