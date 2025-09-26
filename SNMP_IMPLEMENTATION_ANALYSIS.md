# Glances SNMP Support Analysis

## Question: Does the Glances project really have SNMP support?

**Answer: YES, the Glances project has comprehensive SNMP support with full polling capabilities.**

## Executive Summary

The Glances monitoring tool includes a complete SNMP client implementation that serves as a fallback mechanism when the native Glances server is unavailable or when explicitly requested via command-line options. The SNMP support is not just superficial - it includes:

1. **Full SNMP Client Implementation** - Complete SNMP v1/v2c/v3 client with GET and BULK operations
2. **Comprehensive Plugin Support** - Multiple plugins with dedicated SNMP OID definitions
3. **Real Data Polling** - Active polling of system metrics via SNMP from remote hosts
4. **OS-Aware Implementation** - Different SNMP handling for various operating systems

## Detailed Analysis

### Core SNMP Implementation

#### 1. Primary SNMP Client (`glances/snmp.py`)

The main SNMP client implementation provides:

```python
class GlancesSNMPClient:
    """SNMP client class (based on pysnmp library)."""
    
    def get_by_oid(self, *oid):
        """SNMP simple request (list of OID)."""
        # Performs actual SNMP GET operations
        
    def getbulk_by_oid(self, non_repeaters, max_repetitions, *oid):
        """SNMP getbulk request."""
        # Performs SNMP BULK operations for efficient table walking
```

**Key Features:**
- Supports SNMP versions 1, 2c, and 3
- Implements both simple GET and BULK GET requests
- Handles SNMP authentication (community strings and SNMPv3 credentials)
- Provides error handling and result parsing

#### 2. SNMP Statistics Manager (`glances/stats_client_snmp.py`)

High-level manager that coordinates SNMP-based monitoring:

```python
class GlancesStatsClientSNMP(GlancesStats):
    """This class stores, updates and gives stats for the SNMP client."""
    
    def check_snmp(self):
        """Check if SNMP is available on the server."""
        # Tests connectivity using standard system OIDs
        
    def update(self):
        """Update the stats using SNMP."""
        # Coordinates updates across all SNMP-enabled plugins
```

**Capabilities:**
- OS detection via SNMP system description (Linux, Windows, BSD, macOS, etc.)
- Plugin coordination for SNMP-based data collection
- Fallback mechanism when native Glances server fails

### Plugin-Level SNMP Support

#### Plugins with Full SNMP Implementation

1. **Memory Plugin** (`glances/plugins/mem/__init__.py`)
   ```python
   snmp_oid = {
       'default': {
           'total': '1.3.6.1.4.1.2021.4.5.0',    # Total RAM Available
           'free': '1.3.6.1.4.1.2021.4.11.0',    # Total RAM Free
           'shared': '1.3.6.1.4.1.2021.4.13.0',  # Total RAM Shared
           'buffers': '1.3.6.1.4.1.2021.4.14.0', # Total RAM Buffered
           'cached': '1.3.6.1.4.1.2021.4.15.0',  # Total Cached Memory
       }
   }
   ```

2. **Network Plugin** (`glances/plugins/network/__init__.py`)
   ```python
   snmp_oid = {
       'default': {
           'interface_name': '1.3.6.1.2.1.2.2.1.2',  # IF-MIB interface names
           'bytes_recv': '1.3.6.1.2.1.2.2.1.10',     # IF-MIB bytes received
           'bytes_sent': '1.3.6.1.2.1.2.2.1.16',     # IF-MIB bytes sent
       }
   }
   ```

3. **Additional Plugins**: CPU, filesystem, load, system info, uptime - all with specific OID mappings

#### SNMP Polling Process

Each plugin follows this pattern for SNMP data collection:

```python
def update(self):
    """Update stats using the input method."""
    if self.input_method == 'snmp':
        # Get data via SNMP using predefined OIDs
        stats = self.get_stats_snmp(snmp_oid=snmp_oid['default'])
        
        # Process and normalize the SNMP data
        for k in stats:
            stats[k] = int(stats[k]) * 1024  # Convert KB to bytes
            
        # Calculate derived metrics
        stats['used'] = stats['total'] - stats['free']
        stats['percent'] = float(stats['used'] * 100 / stats['total'])
```

### Application Integration

#### Client Connection Logic (`glances/client.py`)

```python
def _login_glances(self):
    """Login to a Glances server"""
    try:
        client_version = self.client.init()
    except OSError as err:
        # Fallback to SNMP when Glances server fails
        self.client_mode = 'snmp'
        logger.error(f"Connection to Glances server failed ({err.errno} {err.strerror})")
        print('No Glances server found. Trying fallback to SNMP...')
```

#### Command Line Support

The application provides comprehensive SNMP command-line options:
- `--snmp-community`: SNMP community string (default: 'public')
- `--snmp-port`: SNMP port (default: 161)
- `--snmp-version`: SNMP version (1, 2c, or 3)
- `--snmp-user`: SNMPv3 username
- `--snmp-auth`: SNMPv3 authentication key
- `--snmp-force`: Force SNMP mode even when Glances server is available

### Standard SNMP MIBs Used

The implementation leverages standard SNMP Management Information Bases (MIBs):

1. **System MIB (RFC 1213)**
   - `1.3.6.1.2.1.1.1.0`: System description
   - `1.3.6.1.2.1.1.5.0`: System hostname

2. **Interface MIB (IF-MIB)**
   - `1.3.6.1.2.1.2.2.1.*`: Network interface statistics

3. **UCD-SNMP-MIB**
   - `1.3.6.1.4.1.2021.4.*`: Memory and system statistics

4. **Host Resources MIB (RFC 2790)**
   - `1.3.6.1.2.1.25.2.3.1.*`: Storage and filesystem information

## Developer Intentions

### Why SNMP Support Was Implemented

Based on the code analysis, the developer implemented SNMP support for several key reasons:

1. **Monitoring Legacy Systems**: Many enterprise systems, network devices, and servers only support SNMP for monitoring
2. **Fallback Mechanism**: When the native Glances server can't be installed or is unavailable
3. **Network Device Monitoring**: Routers, switches, and other network equipment typically only expose metrics via SNMP
4. **Enterprise Integration**: SNMP is a standard protocol in enterprise monitoring environments
5. **Cross-Platform Compatibility**: SNMP works across different operating systems and device types

### Implementation Quality

The SNMP implementation is **production-ready** with:
- Proper error handling and logging
- Support for multiple SNMP versions
- OS-specific adaptations
- Standard MIB compliance
- Bulk request optimization
- Authentication support

## Conclusion

**The Glances project absolutely has real SNMP support with complete polling capabilities.** This is not a stub or placeholder implementation - it's a fully functional SNMP client that:

1. **Actively polls** remote systems using standard SNMP protocols
2. **Retrieves real metrics** including memory, CPU, network, and filesystem data
3. **Processes and normalizes** SNMP data into Glances' internal format
4. **Provides fallback monitoring** when native agents aren't available
5. **Supports enterprise environments** requiring SNMP-only monitoring

The developer's intention was clearly to provide comprehensive monitoring capabilities beyond just the native Glances server-client model, enabling monitoring of any SNMP-enabled system including network devices, legacy servers, and embedded systems.

This implementation represents a significant investment in enterprise-grade monitoring capabilities, making Glances suitable for heterogeneous environments where SNMP is the common monitoring protocol.