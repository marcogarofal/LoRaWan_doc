# Smart Contract Structure - LoRaWAN Roaming System

## **_catalog CATALOG SMART CONTRACT**

### **Data Structures**

```python
class EndpointItem(pt.abi.NamedTuple):
    NetID: pt.abi.Field[pt.abi.String]           # LoRaWAN provider identifier
    Name_Provider: pt.abi.Field[pt.abi.String]   # Provider name
    Endpoint_SC: pt.abi.Field[pt.abi.Uint64]     # Endpoint smart contract ID
```

**Local State (for each registered provider):**
- `NetID`: Unique identifier in LoRaWAN network
- `Name_Provider`: Ecosystem provider name
- `Endpoint_SC`: Provider's endpoint smart contract

### **Main Functions**

**Management Functions:**
- `create()` - Contract creation
- `delete()` - Deletion (creator only)
- `update()` - Update (blocked)
- `clear_state()` - State cleanup

**Provider Management:**
- `opt_in(payment)` - **Provider registration with 1 ALGO payment**
- `close_out()` - Exit from ecosystem

**Provider Operations:**
- `set_entry_provider(NetID, Name, app_id)` - **Set provider data**
- `get_entry_provider(provider)` - **Retrieve provider data**
- `delete_endpoint()` - **Delete provider endpoint**
- `provider_exists(provider)` - **Check provider existence**

---

## **_SC SLA CONTRACT SMART CONTRACT**

### **Data Structures**

```python
class SLA_output(pt.abi.NamedTuple):
    State_SLA: pt.abi.Field[pt.abi.String]      # SLA state (Active/Reject/Banned)
    Token_broker: pt.abi.Field[pt.abi.String]   # Broker token
    GW_id: pt.abi.Field[pt.abi.String]          # Gateway ID

class SLAEntry(pt.abi.NamedTuple):
    NetID: pt.abi.Field[pt.abi.String]                    # Provider ID
    Name_provider: pt.abi.Field[pt.abi.String]            # Provider name
    State_SLA: pt.abi.Field[pt.abi.String]                # Agreement state
    Price_contracting: pt.abi.Field[pt.abi.Uint64]        # Agreed price
    Threshold_contracting: pt.abi.Field[pt.abi.Uint64]    # Packet threshold
    Packet_Counter: pt.abi.Field[pt.abi.Uint64]           # Packet counter
    Token_broker: pt.abi.Field[pt.abi.String]             # Broker token
    GW_id: pt.abi.Field[pt.abi.String]                    # Gateway ID
```

**Global State:**
- `App_ID_catalogo`: Catalog reference
- `Price_SLA`: Base price for SLA
- `Threshold`: Maximum packet threshold
- `Tolerance`: Payment tolerance (default 90%)
- `Payment_Tolerance`: Minimum payment tolerance (default 95%)
- `Token_broker`: Forwarding token
- `GWid`: Gateway identifier

**Box Storage:**
- `sla_entry`: Mapping(String → SLAEntry) for each agreement

### **Main Functions**

**Management Functions:**
- `create()`, `delete()`, `update()`, `clear_state()`
- `opt_in()` - **Blocked (Reject)**

**Initialization & Utility:**
- `init(app_id, price, threshold, tolerance, payment_tolerance, token, gwid)` - **Contract initialization**
- `get_appID_cat()` - Get catalog ID

**SLA Management:**
- `getSLA(type_op, NetID)` - **Retrieve existing SLA**
- `deleteSLA(type_op, NetID)` - **Delete SLA**
- `sla_check(NetID_home, provider, cat_ref, Name_Provider, Endpoint_SC)` - **CORE FUNCTION: Complete SLA management**
- `handshake(app_id_catalogo, provider, price_f, threshold_f)` - **New SLA negotiation**

**Packet & Payment Management:**
- `receive_packet_from_forwarder(NetID_forwarder)` - **Receive and count packets**
- `pay(app_id_catalogo, receiver)` - **Handle automatic payments**

**Admin & Monitoring:**
- `balance(receiver)` - Check account balance
- `ban_home_function(NetID_forwarder)` - **Ban non-compliant provider**
- `get_sla_stats(NetID, type_op)` - **SLA statistics**

---

## **DATA STRUCTURE INTERACTIONS**

### **Main Data Flow:**

1. **Provider Registration** → `EndpointItem` saved in **Catalog Local State**
2. **SLA Negotiation** → `SLAEntry` created in **SLA Contract Box Storage**
3. **Packet Processing** → Update `Packet_Counter` in `SLAEntry`
4. **Payment Trigger** → Reset `Packet_Counter`, execute payment
5. **State Management** → `State_SLA` can become "Active"/"Banned"/"Rejected"

### **Box Storage Keys:**
- `NetID + "_forward_box"` → SLA for outgoing traffic
- `NetID + "_home_box"` → SLA for incoming traffic

### **Possible SLA States:**
- `"Active SLA"` → Active and functioning agreement
- `"SLA rejected"` → Agreement rejected during negotiation
- `"Banned"` → Provider banned for non-compliance
- `"Accept"/"Reject"` → Temporary responses for single packets


### **Workflow:**

1. **Registration**: Providers register in the **Catalog** by paying 1 ALGO
2. **SLA Negotiation**: When roaming is needed, the **SLA Contract** negotiates agreements between providers
3. **Packet Forwarding**: Packets are forwarded and counted
4. **Payment**: When threshold is reached, automatic payments are executed
5. **Monitoring**: The system monitors and can ban providers who don't respect agreements

The **Catalog** is the central registry, while the **SLA Contract** manages the business logic of LoRaWAN roaming with automated payments based on traffic thresholds.
