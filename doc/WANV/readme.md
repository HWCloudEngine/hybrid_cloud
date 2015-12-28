* Transparent
    - NAT is done by gateway device (e.g. DSL modem/router)
* Stickiness
    - Flows remain on selected link and are not             re-routed
* User Experience
    - Best fitting link selected for each flow, based on round robin on available links
    - Queuing discipline guarantees fairness and prevents bandwidth hogging
    - Link based internet service delay measurement method 
* Path Balancing
    - Optimize flow distribution over all links
    - Passive learning algorithm for long-term link utilization optimization

## Multi-ended
* Secured Cross-Branch Tunneling
    - Paths between branches use tunneling protocols (e.g. GRE, GRE over IPSEC)
* Efficient
    - WANV IP addresses are public, NAT is not done between branches
* High Fidelity (site-to-site)
    - Original packets not modified
    - Internal IP addresses are retained
* User Experience
    - Flows are transparently re-routed to better paths in real-time
    - Path performance is monitored in real-time (state, delay, delay variation, packet loss)
    - Path protection with no noticeable interruption to ongoing user sessions
    - Queuing discipline guarantees fairness and prevents bandwidth hogging
    - Online hybrid model to predict network performance in real time
* Path Balancing
    - Optimize flow distribution over all paths

