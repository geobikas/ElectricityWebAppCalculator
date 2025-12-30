import streamlit as st
from datetime import datetime

# --- ΣΥΝΑΡΤΗΣΕΙΣ ΥΠΟΛΟΓΙΣΜΟΥ ---
def calculate_yko(kwh, k1_l, k2_l, p1, p2, p3):
    """Υπολογίζει τις χρεώσεις ΥΚΩ ανά κλίμακα και επιστρέφει πλήρη ανάλυση."""
    k1 = min(kwh, k1_l)
    r = max(0, kwh - k1)
    k2 = min(r, k2_l)
    k3 = max(0, r - k2)
    return {
        "k1_kwh": k1, "k1_cost": round(k1 * p1, 2),
        "k2_kwh": k2, "k2_cost": round(k2 * p2, 2),
        "k3_kwh": k3, "k3_cost": round(k3 * p3, 2),
        "total": round((k1 * p1) + (k2 * p2) + (k3 * p3), 2)
    }

# --- ΡΥΘΜΙΣΕΙΣ ΣΕΛΙΔΑΣ ---
st.set_page_config(page_title="Energy Pro Analysis", layout="wide", page_icon="⚡")

st.title("⚡ Ολοκληρωμένη Ανάλυση Λογαριασμού Ρεύματος")
st.markdown("---")

# --- ΔΗΜΙΟΥΡΓΙΑ TABS ---
tab_calc, tab_settings = st.tabs(["🧮 Υπολογισμός & Αναφορά", "⚙️ Ρυθμίσεις Παραμέτρων"])

# --- TAB SETTINGS: ΠΑΡΑΜΕΤΡΟΠΟΙΗΣΗ ---
with tab_settings:
    st.header("Ρυθμίσεις Σταθερών Χρεώσεων")
    st.info("Εδώ ορίζετε τις τιμές που παραμένουν σταθερές στον λογαριασμό σας.")
    
    col_s1, col_s2, col_s3 = st.columns(3)
    
    with col_s1:
        st.subheader("Ενέργεια & Ισχύς")
        p_kwh_day = st.number_input("Κόστος kWh Ημέρας (€)", value=0.1049, format="%.4f")
        p_kwh_night = st.number_input("Κόστος kWh Νύχτας (€)", value=0.1049, format="%.4f")
        p_fixed = st.number_input("Μηνιαίο Πάγιο (€)", value=7.90)
        p_kva = st.selectbox("Ισχύς kVa", [8, 12, 25], index=0)
        p_vat = st.slider("ΦΠΑ (%)", 0, 24, 6) / 100

    with col_s2:
        st.subheader("Δήμος & Τέλη (από εικόνα)")
        p_sqm = st.number_input("Τετραγωνικά Μέτρα (Τ.Μ.)", value=87)
        p_dt = st.number_input("Συντελεστής ΔΤ (€/τμ)", value=1.85, format="%.4f")
        p_df = st.number_input("Συντελεστής ΦΤ/ΔΦ (€/τμ)", value=0.07, format="%.4f")
        p_ert = st.number_input("ΕΡΤ (Ετήσια €)", value=36.0)

    with col_s3:
        st.subheader("ΤΑΠ & Παλαιότητα")
        p_tap_zone = st.number_input("ΤΑΠ (Τιμή Ζώνης)", value=1000)
        p_age = st.number_input("Συντ. Παλαιότητας", value=0.65)
        p_tap_coeff = st.number_input("Συντ. ΤΑΠ", value=0.00035, format="%.5f")

# --- TAB CALCULATION: ΥΠΟΛΟΓΙΣΜΟΣ & ΑΝΑΛΥΤΙΚΗ ΑΝΑΦΟΡΑ ---
with tab_calc:
    col_in1, col_in2 = st.columns(2)
    with col_in1:
        d_start = st.date_input("Έναρξη Περιόδου", datetime(2025, 12, 29))
        m1_day = st.number_input("Παλιά Ένδειξη Ημέρας", value=0.0)
        m1_night = st.number_input("Παλιά Ένδειξη Νύχτας", value=0.0)
    
    with col_in2:
        d_end = st.date_input("Λήξη Περιόδου", datetime(2025, 12, 30))
        m2_day = st.number_input("Νέα Ένδειξη Ημέρας", value=0.0)
        m2_night = st.number_input("Νέα Ένδειξη Νύχτας", value=0.0)

    days = (d_end - d_start).days
    if days <= 0:
        st.error("⚠️ Η ημερομηνία λήξης πρέπει να είναι μεταγενέστερη της έναρξης.")
        st.stop()
        
    kwh_day = max(0.0, m2_day - m1_day)
    kwh_night = max(0.0, m2_night - m1_night)
    total_kwh = kwh_day + kwh_night
    day_ratio = days / 365

    # 1. Προμήθεια
    cost_fixed = round(p_fixed * days / 30, 2)
    cost_en_day = round(kwh_day * p_kwh_day, 2)
    cost_en_night = round(kwh_night * p_kwh_night, 2)
    supply_total = round(cost_fixed + cost_en_day + cost_en_night, 2)

    # 2. Ρυθμιζόμενες (Εκτός ΥΚΩ)
    cost_admie = round(total_kwh * 0.00999, 2)
    cost_deddie = round((p_kva * 6.21 * day_ratio) + (total_kwh * 0.00339), 2)
    cost_etmear = round(total_kwh * 0.017, 2)

    # 3. ΥΚΩ (με τις κλίμακες)
    k1_l = round(1600 * days / 120)
    k2_l = round(400 * days / 120)
    yko_h = calculate_yko(kwh_day, k1_l, k2_l, 0.0069, 0.0500, 0.0850)
    yko_n = calculate_yko(kwh_night, k1_l, k2_l, 0.0069, 0.0150, 0.0300)
    yko_total = round(yko_h["total"] + yko_n["total"], 2)
    
    reg_total = round(cost_admie + cost_deddie + cost_etmear + yko_total, 2)

    # 4. Φόροι & Δήμος
    efk = 1.00 
    det_base = supply_total + reg_total + efk
    det_5mil = round(det_base * 0.005, 2)
    vat_val = round(det_base * p_vat, 2)
    cost_dt = round(p_sqm * p_dt * day_ratio, 2)
    cost_df = round(p_sqm * p_df * day_ratio, 2)
    cost_tap = round(p_sqm * p_tap_zone * p_age * p_tap_coeff * day_ratio, 2)
    cost_ert = round((p_ert * days) / 365, 2)

    total_bill = round(det_base + vat_val + det_5mil + cost_dt + cost_df + cost_tap + cost_ert, 2)

    # --- ΑΠΟΤΕΛΕΣΜΑΤΑ ---
    st.divider()
    st.metric(label="✅ ΣΥΝΟΛΙΚΟ ΠΟΣΟ ΠΛΗΡΩΜΗΣ", value=f"{total_bill:.2f} €")

    # ΕΝΟΤΗΤΑ 1: ΠΡΟΜΗΘΕΙΑ
    st.subheader("1. Ανάλυση Προμήθειας")
    st.table({
        "Περιγραφή": ["Πάγιο", "Ενέργεια Ημέρας", "Ενέργεια Νύχτας", "ΣΥΝΟΛΟ"],
        "Ποσότητα": [f"{days} ημέρες", f"{kwh_day:.1f} kWh", f"{kwh_night:.1f} kWh", "-"],
        "Τιμή Μονάδας": [f"{p_fixed} €/μήνα", f"{p_kwh_day:.4f} €/kWh", f"{p_kwh_night:.4f} €/kWh", "-"],
        "Σύνολο (€)": [cost_fixed, cost_en_day, cost_en_night, supply_total]
    })

    # ΕΝΟΤΗΤΑ 2: ΡΥΘΜΙΖΟΜΕΝΕΣ
    st.subheader("2. Ρυθμιζόμενες Χρεώσεις & ΥΚΩ")
    col_a, col_b = st.columns(2)
    with col_a:
        st.write(f"**ΑΔΜΗΕ:** {cost_admie:.2f} €")
        st.write(f"**ΔΕΔΔΗΕ:** {cost_deddie:.2f} €")
        st.write(f"**ΕΤΜΕΑΡ:** {cost_etmear:.2f} €")
        st.write(f"**ΥΚΩ (Σύνολο):** {yko_total:.2f} €")
    
    with col_b:
        with st.expander("🔎 Ανάλυση Κλιμάκων ΥΚΩ (kWh)"):
            st.markdown("**ΥΚΩ Ημέρας**")
            st.text(f"0-{k1_l} kWh: {yko_h['k1_kwh']:.1f} kWh x 0.0069 = {yko_h['k1_cost']:.2f}€")
            if yko_h['k2_kwh'] > 0: st.text(f"{k1_l}-{k1_l+k2_l} kWh: {yko_h['k2_kwh']:.1f} kWh x 0.0500 = {yko_h['k2_cost']:.2f}€")
            if yko_h['k3_kwh'] > 0: st.text(f"> {k1_l+k2_l} kWh: {yko_h['k3_kwh']:.1f} kWh x 0.0850 = {yko_h['k3_cost']:.2f}€")
            
            if kwh_night > 0:
                st.markdown("**ΥΚΩ Νύχτας**")
                st.text(f"0-{k1_l} kWh: {yko_n['k1_kwh']:.1f} kWh x 0.0069 = {yko_n['k1_cost']:.2f}€")
                if yko_n['k2_kwh'] > 0: st.text(f"{k1_l}-{k1_l+k2_l} kWh: {yko_n['k2_kwh']:.1f} kWh x 0.0150 = {yko_n['k2_cost']:.2f}€")
                if yko_n['k3_kwh'] > 0: st.text(f"> {k1_l+k2_l} kWh: {yko_n['k3_kwh']:.1f} kWh x 0.0300 = {yko_n['k3_cost']:.2f}€")

    # ΕΝΟΤΗΤΑ 3: ΦΟΡΟΙ & ΔΗΜΟΣ
    st.subheader("3. Λοιπές Χρεώσεις & Φόροι")
    st.table({
        "Κατηγορία": ["ΦΠΑ", "Ε.Φ.Κ.", "Ειδικό Τέλος 5‰", "Δήμος (ΔΤ+ΔΦ)", "Τ.Α.Π.", "ΕΡΤ"],
        "Ανάλυση": [f"{p_vat*100:.0f}% επί Φορολογητέου", "Σταθερή χρέωση", "0.5% επί Φορολογητέου", f"{p_sqm} τ.μ. x συντελεστές", "Βάσει Ζώνης & Παλαιότητας", f"Αναλογία {days} ημερών"],
        "Ποσό (€)": [vat_val, efk, det_5mil, round(cost_dt+cost_df, 2), cost_tap, cost_ert]
    })
