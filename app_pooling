import streamlit as st
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.animation as animation

# Set up browser tab configuration
st.set_page_config(page_title="Queueing Simulation", layout="wide")

st.title("🏟️ Specialized Lines vs. Cross-Trained Queueing System")
st.markdown("""
This web application simulates a service environment processing **Standard** and **Complex** tasks. 
Adjust parameters in the sidebar to test how process layouts respond to operational variability and switching penalties.
""")

# ========================================================
# 🎛️ SIDEBAR INPUT CONTROLS (Replaces ipywidgets layout)
# ========================================================
st.sidebar.header("🛠️ System Configuration")

mean_arr = st.sidebar.slider("Mean Arrival (min):", 1.0, 20.0, 4.0, step=0.5)
mean_std = st.sidebar.slider("Mean Standard Service (min):", 1.0, 20.0, 5.0, step=0.5)
mean_cpx = st.sidebar.slider("Mean Complex Service (min):", 1.0, 30.0, 15.0, step=0.5)
setup_penalty = st.sidebar.slider("Setup/Switching Penalty (min):", 0.0, 10.0, 2.0, step=0.5)

# High-contrast structure selector
queue_type = st.radio(
    "Select Branch Structure Architecture:",
    options=['Pooled (Cross-Trained Staff)', 'Unpooled (Specialized Staff)']
)
is_pooled = (queue_type == 'Pooled (Cross-Trained Staff)')

def convert_to_lognormal_params(mean, std):
    if mean <= 0 or std <= 0: return 0.0, 0.1
    variance = (std if std > 0 else 0.1) ** 2
    mu_log = np.log((mean ** 2) / np.sqrt((mean ** 2) + variance))
    sigma_log = np.sqrt(np.log(1.0 + (variance / (mean ** 2))))
    return mu_log, sigma_log

# Create placeholder containers to control where outputs render on the web page
visual_placeholder = st.empty()
analytics_placeholder = st.empty()

# ========================================================
# 🎬 SECTION 1: NATIVE WEB ANIMATION PLAYER
# ========================================================
if st.button("🎬 Build Animation Player", type="secondary"):
    with visual_placeholder.container():
        st.subheader("Live Process Flow Visualization")
        
        NUM_CUSTOMERS = 40  
        PROP_STANDARD = 0.80
        
        # Seed locked for direct comparative visual evaluation
        np.random.seed(42) 
        inter_arrivals = np.random.lognormal(np.log(mean_arr) - 0.02, 0.2, NUM_CUSTOMERS)
        arrival_times = np.cumsum(inter_arrivals)
        job_types = ['standard' if np.random.rand() < PROP_STANDARD else 'complex' for _ in range(NUM_CUSTOMERS)]
        service_times = [np.random.lognormal(np.log(mean_std) - 0.02, 0.2) if jt == 'standard' else np.random.lognormal(np.log(mean_cpx) - 0.08, 0.4) for jt in job_types]
        np.random.seed(None) 
        
        teller_free_time = [0.0, 0.0]
        teller_last_job = [None, None]  
        customer_states = []
        
        for idx in range(NUM_CUSTOMERS):
            arr = arrival_times[idx]
            jt = job_types[idx]
            ser = service_times[idx]
            
            chosen_teller = np.argmin(teller_free_time) if is_pooled else (0 if jt == 'standard' else 1)
            
            actual_setup = 0.0
            if is_pooled and teller_last_job[chosen_teller] is not None and teller_last_job[chosen_teller] != jt:
                actual_setup = setup_penalty
                
            start_ser = max(arr, teller_free_time[chosen_teller])
            end_ser = start_ser + ser + actual_setup
            
            teller_free_time[chosen_teller] = end_ser
            teller_last_job[chosen_teller] = jt  
            
            customer_states.append({
                'arrival': arr, 'start_service': start_ser, 'end_service': end_ser, 
                'teller': chosen_teller, 'type': jt, 'had_setup': (actual_setup > 0)
            })
            
        total_simulation_duration = arrival_times[-1] + 10.0
        time_steps = np.linspace(0, total_simulation_duration, 100)
        
        fig, ax = plt.subplots(figsize=(9, 4.2))
        
        def update_frame(frame_idx):
            ax.clear()
            ax.set_xlim(-1, 11)
            ax.set_ylim(-1, 6)
            ax.axis('off')
            
            current_time = time_steps[frame_idx]
            ax.text(5, 5.5, f"Live Flow: {queue_type}", ha='center', fontsize=12, fontweight='bold', color='#1A5276' if is_pooled else '#2E4053')
            ax.text(0, 5.0, f"Time: {current_time:.1f} min", fontsize=11, family='monospace', fontweight='bold', color='#E74C3C')
            
            t1_busy, t2_busy, t1_setup, t2_setup = False, False, False, False
            pooled_q_count, unpooled_q1_count, unpooled_q2_count = 0, 0, 0
            
            for c in customer_states:
                color_code = '#2980B9' if c['type'] == 'standard' else '#8E44AD'
                if c['start_service'] <= current_time <= c['end_service']:
                    is_in_setup = c['had_setup'] and (current_time <= (c['start_service'] + setup_penalty))
                    if c['teller'] == 0:
                        t1_busy, t1_setup = True, is_in_setup
                        ax.add_patch(plt.Circle((8.6, 3.8), 0.24, facecolor=color_code, edgecolor='black', zorder=5))
                    else:
                        t2_busy, t2_setup = True, is_in_setup
                        ax.add_patch(plt.Circle((8.6, 1.4), 0.24, facecolor=color_code, edgecolor='black', zorder=5))
                elif c['arrival'] <= current_time < c['start_service']:
                    if is_pooled:
                        cx, cy = 6.5 - (pooled_q_count * 0.48), 2.6
                        pooled_q_count += 1
                    else:
                        if c['teller'] == 0:
                            cx, cy = 6.5 - (unpooled_q1_count * 0.48), 3.8
                            unpooled_q1_count += 1
                        else:
                            cx, cy = 6.5 - (unpooled_q2_count * 0.48), 1.4
                            unpooled_q2_count += 1
                    ax.add_patch(plt.Circle((cx, cy), 0.18, facecolor=color_code, edgecolor='black', alpha=0.8, zorder=4))
            
            c1 = '#E67E22' if t1_setup else ('#E74C3C' if t1_busy else '#2ECC71')
            c2 = '#E67E22' if t2_setup else ('#E74C3C' if t2_busy else '#2ECC71')
            ax.add_patch(plt.Rectangle((8, 3.2), 1.2, 1.2, facecolor=c1, edgecolor='black', linewidth=1.5))
            ax.text(8.6, 3.8, "T1\n(Setup)" if t1_setup else "T1", ha='center', va='center', color='white', weight='bold')
            ax.add_patch(plt.Rectangle((8, 0.8), 1.2, 1.2, facecolor=c2, edgecolor='black', linewidth=1.5))
            ax.text(8.6, 1.4, "T2\n(Setup)" if t2_setup else "T2", ha='center', va='center', color='white', weight='bold')
            
            ax.text(4.5, 3.8, "Line 1 (Standard)" if not is_pooled else "Shared Central Queue Layout", ha='center', fontsize=9, weight='bold', color='#7F8C8D')
            if not is_pooled: ax.text(4.5, 1.4, "Line 2 (Complex)", ha='center', fontsize=9, weight='bold', color='#7F8C8D')

        anim = animation.FuncAnimation(fig, update_frame, frames=len(time_steps), interval=120, repeat=False)
        
        # Convert Matplotlib animation directly into HTML5 video player layout native to web browsers
        st.components.v1.html(anim.to_jshtml(), height=550, scrolling=False)
        plt.close(fig)

# ========================================================
# 📊 SECTION 2: 2-YEAR HISTOGRAM ANALYTICS ENGINE
# ========================================================
if st.button("📊 Run 2-Year Analytics", type="primary"):
    with analytics_placeholder.container():
        st.subheader("Long-Term Horizon Analytical Profile")
        
        # Lock the random seed inside analytics engine so all 40 students see identical stable metrics
        np.random.seed(42)
        
        SIM_TIME_LIMIT = 2 * 52 * 5 * 8 * 60  
        PROP_STANDARD = 0.80
        
        mu_arr, sig_arr = convert_to_lognormal_params(mean_arr, mean_arr * 0.25)
        mu_std, sig_std = convert_to_lognormal_params(mean_std, mean_std * 0.2)
        mu_cpx, sig_cpx = convert_to_lognormal_params(mean_cpx, mean_cpx * 0.4)
        
        teller_free_time = [0.0, 0.0]
        teller_busy_time = [0.0, 0.0]
        teller_last_job = [None, None]
        waiting_times = []
        current_arrival_clock = 0.0
        
        while current_arrival_clock < SIM_TIME_LIMIT:
            inter_arrival = np.random.lognormal(mu_arr, sig_arr)
            current_arrival_clock += max(inter_arrival, 0.1)
            if current_arrival_clock >= SIM_TIME_LIMIT: break
                
            job_type = 'standard' if np.random.rand() < PROP_STANDARD else 'complex'
            service = np.random.lognormal(mu_std, sig_std) if job_type == 'standard' else np.random.lognormal(mu_cpx, sig_cpx)
            service = max(service, 0.1)
            
            chosen_teller = np.argmin(teller_free_time) if is_pooled else (0 if job_type == 'standard' else 1)
            
            actual_setup = 0.0
            if is_pooled and teller_last_job[chosen_teller] is not None and teller_last_job[chosen_teller] != job_type:
                actual_setup = setup_penalty
                
            total_processing_duration = service + actual_setup
            wait = 0.0 if current_arrival_clock >= teller_free_time[chosen_teller] else teller_free_time[chosen_teller] - current_arrival_clock
            start_service = max(current_arrival_clock, teller_free_time[chosen_teller])
            
            waiting_times.append(wait)
            teller_free_time[chosen_teller] = start_service + total_processing_duration
            teller_last_job[chosen_teller] = job_type
            
            if start_service < SIM_TIME_LIMIT:
                teller_busy_time[chosen_teller] += min(total_processing_duration, SIM_TIME_LIMIT - start_service)

        waiting_times = np.array(waiting_times)
        utilizations = [(busy / SIM_TIME_LIMIT) * 100 for busy in teller_busy_time]
        avg_wait = np.mean(waiting_times)
        avg_utilization = np.mean(utilizations)
        
        if avg_utilization >= 100.0 or avg_wait > 500:
            st.error("💥 SYSTEM COLLAPSED / UNSTABLE!")
            st.write(f"Due to the {setup_penalty} min switching penalty, total system utilization hit **{avg_utilization:.1f}%**.")
            st.write("The queue size grew infinitely. This proves pooling backfires when switching costs cross the threshold!")
        else:
            fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 4.5))
            
            ax1.hist(waiting_times, bins=50, color='#1A5276' if is_pooled else '#2E4053', edgecolor='black', alpha=0.75, density=True)
            ax1.axvline(avg_wait, color='#C0392B', linestyle='--', linewidth=2, label=f'Mean Wait: {avg_wait:.2f} min')
            ax1.set_title("Customer Waiting Time Distribution", fontweight='bold')
            ax1.set_xlabel("Wait Time (Minutes)")
            ax1.grid(True, linestyle=':', alpha=0.5)
            ax1.legend()
            
            labels = ['Teller 1 (Standard)', 'Teller 2 (Complex)', 'System Avg'] if not is_pooled else ['Teller 1 (Cross)', 'Teller 2 (Cross)', 'System Avg']
            bars = ax2.bar(labels, utilizations + [avg_utilization], color=['#2980B9', '#8E44AD', '#D35400'], edgecolor='black', width=0.4)
            ax2.set_title("Staff Allocation & Capacity Utilization Analysis", fontweight='bold')
            ax2.set_ylabel("Utilization Level (%)")
            ax2.set_ylim(0, 115)
            ax2.grid(axis='y', linestyle=':', alpha=0.5)
            for bar in bars:
                h = bar.get_height()
                ax2.text(bar.get_x() + bar.get_width()/2., h + 2, f'{h:.1f}%', ha='center', va='bottom', weight='bold')
                
            st.pyplot(fig)
            
            # Print performance specs in a clean web panel
            st.info(f"""
            📊 **Performance Dashboard Metrics Summary:**
            * **Structure Model Architecture:** {queue_type.upper()}
            * **Total Transactions Processed Over 2 Years:** {len(waiting_times):,} customers
            * **Combined Branch Average Staff Utilization Rate:** {avg_utilization:.1f}% *(Includes setup overhead)*
            * **Average Customer Waiting Time Duration:** **{avg_wait:.2f} minutes**
            """)
