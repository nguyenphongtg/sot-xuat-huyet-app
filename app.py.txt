import streamlit as st
import pandas as pd
from datetime import datetime

# Cấu hình trang web
st.set_page_config(page_title="Công cụ Phòng chống Sốt xuất huyết", page_icon="🦟", layout="wide")

st.title("🦟 Hệ thống Quản lý và Phòng chống Sốt xuất huyết")
st.write("Công cụ hỗ trợ cán bộ y tế tính toán chỉ số véc-tơ, định mức hóa chất và quản lý điều tra vãng gia.")

# Tạo các Tab chức năng
tab1, tab2, tab3 = st.tabs([
    "📊 1. Tính chỉ số véc-tơ & Hóa chất", 
    "🏠 2. Điều tra vãng gia hộ gia đình", 
    "📁 3. Quản lý Biên bản & Mẫu biểu"
])

# --- TAB 1: TÍNH CHỈ SỐ VÉC-TƠ & HÓA CHẤT ---
with tab1:
    st.header("1. Tính toán chỉ số véc-tơ lăng quăng (HI, CI, BI)")
    col1, col2 = st.columns(2)
    with col1:
        total_houses = st.number_input("Tổng số nhà được kiểm tra", min_value=1, value=100)
        houses_with_larvae = st.number_input("Số nhà phát hiện có lăng quăng", min_value=0, value=15)
    with col2:
        total_containers = st.number_input("Tổng số dụng cụ chứa nước kiểm tra", min_value=1, value=300)
        containers_with_larvae = st.number_input("Số dụng cụ có lăng quăng", min_value=0, value=20)
    
    if st.button("Tính toán chỉ số véc-tơ", type="primary"):
        hi = (houses_with_larvae / total_houses) * 100
        ci = (containers_with_larvae / total_containers) * 100
        bi = (containers_with_larvae / total_houses) * 100
        
        st.divider()
        res1, res2, res3 = st.columns(3)
        res1.metric("Chỉ số nhà (HI)", f"{hi:.2f}%")
        res2.metric("Chỉ số dụng cụ (CI)", f"{ci:.2f}%")
        res3.metric("Chỉ số Breteau (BI)", f"{bi:.2f}")
        
        if bi >= 20:
            st.error("⚠️ **CẢNH BÁO:** Chỉ số BI >= 20, khu vực có nguy cơ cao bùng phát dịch sốt xuất huyết!")
        else:
            st.success("✅ Chỉ số ở ngưỡng an toàn/nguy cơ thấp. Tiếp tục theo dõi.")

    st.divider()
    st.header("2. Tính định mức hóa chất phun")
    method = st.selectbox("Phương pháp phun:", ["Phun tồn lưu (Tường/vách - m²)", "Phun không gian / ULV (Thể tích - m³)"])
    
    c1, c2, c3 = st.columns(3)
    with c1:
        length = st.number_input("Chiều dài phòng (m)", min_value=0.1, value=5.0)
    with c2:
        width = st.number_input("Chiều rộng phòng (m)", min_value=0.1, value=4.0)
    with c3:
        height = st.number_input("Chiều cao phòng (m)", min_value=0.1, value=3.0)
        
    if "tồn lưu" in method:
        dose = st.number_input("Liều lượng khuyến cáo (g/m² hoặc ml/m²)", min_value=0.001, value=0.050, format="%.3f")
        if st.button("Tính hóa chất phun tồn lưu"):
            wall_area = 2 * (length + width) * height
            total_chem = wall_area * dose
            st.info(f"📐 **Diện tích tường cần phun:** {wall_area:.2f} m²")
            st.success(f"🧪 **Lượng hóa chất cần dùng:** **{total_chem:.2f}** gram (hoặc ml)")
    else:
        dose = st.number_input("Liều lượng khuyến cáo (ml/m³)", min_value=0.001, value=0.020, format="%.3f")
        if st.button("Tính hóa chất phun không gian"):
            volume = length * width * height
            total_chem = volume * dose
            st.info(f"📦 **Thể tích không gian:** {volume:.2f} m³")
            st.success(f"🧪 **Lượng hóa chất cần dùng:** **{total_chem:.2f}** ml")

# --- TAB 2: ĐIỀU TRA VÃNG GIA ---
with tab2:
    st.header("Nhập phiếu điều tra vãng gia hộ gia đình")
    with st.form("survey_form"):
        col_a, col_b = st.columns(2)
        with col_a:
            owner_name = st.text_input("Họ và tên chủ hộ")
            address = st.text_input("Địa chỉ / Số nhà / Tổ")
            survey_date = st.date_input("Ngày kiểm tra", value=datetime.now())
        with col_b:
            num_containers = st.number_input("Tổng số dụng cụ chứa nước", min_value=0, value=3)
            has_larvae = st.selectbox("Phát hiện lăng quăng?", ["Không", "Có"])
            note = st.text_area("Biện pháp xử lý tại chỗ / Ghi chú")
            
        submitted = st.form_submit_button("Lưu thông tin hộ gia đình")
        if submitted:
            st.success(f"✅ Đã ghi nhận thông tin hộ: **{owner_name}** - Địa chỉ: {address}")

# --- TAB 3: QUẢN LÝ BIÊN BẢN ---
with tab3:
    st.header("Quản lý & Lưu trữ Biên bản")
    st.subheader("📤 Tải lên biên bản/báo cáo đã ký")
    uploaded_file = st.file_uploader("Chọn file biên bản (PDF, PNG, JPG, DOCX)", type=["pdf", "png", "jpg", "docx"])
    if uploaded_file is not None:
        st.success(f"🎉 Đã lưu trữ thành công file: **{uploaded_file.name}**")