# SNMP Analysis for Glances Repository

## Overview

This document provides a comprehensive analysis of all SNMP-related functionality in the Glances monitoring tool. Glances supports SNMP as a fallback method for collecting system statistics when the native Glances server is not available or when explicitly forced via command-line options.

## Core SNMP Implementation Files

### 1. `glances/snmp.py` - Core SNMP Client

**Purpose**: Main SNMP client implementation using the PySNMP library.

**Key Components**:
- `GlancesSNMPClient` class: Primary SNMP client implementation
- Supports SNMP versions 1, 2c, and 3
- Implements both simple OID requests and bulk requests

**Key Methods**:
- `__init__(host, port, version, community, user, auth)`: Initialize SNMP client with connection parameters
- `get_by_oid(*oid)`: Execute simple SNMP GET request for specific OIDs
- `getbulk_by_oid(non_repeaters, max_repetitions, *oid)`: Execute SNMP GETBULK request for efficient table walking
- `__buid_result(varBinds)`: Build result dictionary from SNMP response
- `__get_result__(errorIndication, errorStatus, errorIndex, varBinds)`: Handle SNMP response and errors

**SNMP Version Support**:
- **Version 1**: Basic functionality, no bulk requests
- **Version 2c**: Default version with community-based authentication and bulk request support
- **Version 3**: Enhanced security with user-based authentication

**Dependencies**: 
- Required: `pysnmp-lextudio` package
- Fails with critical error if PySNMP library not found

### 2. `glances/stats_client_snmp.py` - SNMP Statistics Client

**Purpose**: High-level SNMP client that manages statistics collection from SNMP servers.

**Key Components**:
- `GlancesStatsClientSNMP` class: Extends `GlancesStats` for SNMP-based monitoring
- OS detection mechanism using SNMP system information
- Plugin management for SNMP-based data collection

**Key Methods**:
- `check_snmp()`: Verify SNMP server connectivity and detect OS type
- `get_system_name(oid_system_name)`: Map SNMP system description to short OS names
- `update()`: Coordinate stats updates across all plugins using SNMP

**OS Detection Mapping**:
```python
oid_to_short_system_name = {
    '.*Linux.*': 'linux',
    '.*Darwin.*': 'mac', 
    '.*BSD.*': 'bsd',
    '.*Windows.*': 'windows',
    '.*Cisco.*': 'cisco',
    '.*VMware ESXi.*': 'esxi',
    '.*NetApp.*': 'netapp',
}
```

**Standard OIDs Used**:
- `1.3.6.1.2.1.1.5.0`: System hostname
- `1.3.6.1.2.1.1.1.0`: System description for OS detection

### 3. `glances/plugins/plugin/model.py` - Plugin Model with SNMP Support

**Purpose**: Base plugin model that provides SNMP functionality to all plugins.

**Key Methods**:
- `get_stats_snmp(bulk=False, snmp_oid=None)`: Generic SNMP data retrieval method used by all plugins
- `short_system_name` property: OS-specific behavior switching

**SNMP Request Types**:
- **Simple GET**: For single or multiple specific OIDs
- **BULK GET**: For efficient table walking (SNMP v2c+ only)

## Plugin-Specific SNMP Implementations

### Network Plugin (`glances/plugins/network/__init__.py`)

**SNMP OIDs**:
```python
snmp_oid = {
    'default': {
        'interface_name': '1.3.6.1.2.1.2.2.1.2',  # ifDescr
        'bytes_recv': '1.3.6.1.2.1.2.2.1.10',    # ifInOctets
        'bytes_sent': '1.3.6.1.2.1.2.2.1.16',    # ifOutOctets
    }
}
```
**Purpose**: Monitor network interface statistics using standard IF-MIB

### Memory Plugin (`glances/plugins/mem/__init__.py`)

**SNMP OIDs**:
```python
snmp_oid = {
    'default': {
        'total': '1.3.6.1.4.1.2021.4.5.0',    # memTotalReal
        'free': '1.3.6.1.4.1.2021.4.11.0',    # memAvailReal  
        'shared': '1.3.6.1.4.1.2021.4.13.0',  # memShared
        'buffers': '1.3.6.1.4.1.2021.4.14.0', # memBuffer
        'cached': '1.3.6.1.4.1.2021.4.15.0',  # memCached
    }
    # Special handling for Windows/ESXi in filesystem table
}
```
**Purpose**: Monitor memory usage using NET-SNMP UCD-SNMP-MIB

### CPU Plugin (`glances/plugins/cpu/__init__.py`)

**SNMP OIDs**:
```python
snmp_oid = {
    'default': {
        'user': '1.3.6.1.4.1.2021.11.9.0',    # ssCpuUser
        'system': '1.3.6.1.4.1.2021.11.10.0', # ssCpuSystem  
        'idle': '1.3.6.1.4.1.2021.11.11.0',   # ssCpuIdle
    },
    'windows': {
        'percent': '1.3.6.1.2.1.25.3.3.1.2',  # hrProcessorLoad
    }
}
```
**Purpose**: Monitor CPU utilization with OS-specific implementations

### Filesystem Plugin (`glances/plugins/fs/__init__.py`)

**SNMP OIDs**:
```python
snmp_oid = {
    'default': {
        'mnt_point': '1.3.6.1.4.1.2021.9.1.2',  # dskPath
        'device_name': '1.3.6.1.4.1.2021.9.1.3', # dskDevice
        'size': '1.3.6.1.4.1.2021.9.1.6',       # dskTotal
        'used': '1.3.6.1.4.1.2021.9.1.8',       # dskUsed
        'percent': '1.3.6.1.4.1.2021.9.1.9',    # dskPercent
    },
    'windows': {
        'mnt_point': '1.3.6.1.2.1.25.2.3.1.3',  # hrStorageDescr
        'alloc_unit': '1.3.6.1.2.1.25.2.3.1.4', # hrStorageAllocationUnits
        'size': '1.3.6.1.2.1.25.2.3.1.5',       # hrStorageSize
        'used': '1.3.6.1.2.1.25.2.3.1.6',       # hrStorageUsed
    }
}
```
**Purpose**: Monitor filesystem usage with platform-specific MIBs

### System Plugin (`glances/plugins/system/__init__.py`)

**SNMP OIDs**:
```python
snmp_oid = {
    'default': {
        'hostname': '1.3.6.1.2.1.1.5.0',    # sysName
        'system_name': '1.3.6.1.2.1.1.1.0'  # sysDescr
    },
    'netapp': {
        'hostname': '1.3.6.1.2.1.1.5.0',
        'system_name': '1.3.6.1.2.1.1.1.0', 
        'platform': '1.3.6.1.4.1.789.1.1.5.0'  # NetApp-specific
    }
}
```
**Purpose**: Gather basic system information

### Load Plugin (`glances/plugins/load/__init__.py`)

**SNMP OIDs**:
```python
snmp_oid = {
    'min1': '1.3.6.1.4.1.2021.10.1.3.1',   # laLoad.1
    'min5': '1.3.6.1.4.1.2021.10.1.3.2',   # laLoad.5  
    'min15': '1.3.6.1.4.1.2021.10.1.3.3',  # laLoad.15
}
```
**Purpose**: Monitor system load averages

### Additional Plugins with SNMP Support

The following plugins also have SNMP input method support but may not have dedicated OID definitions:

- **IRQ Plugin** (`glances/plugins/irq/__init__.py`)
- **IP Plugin** (`glances/plugins/ip/__init__.py`) 
- **Connections Plugin** (`glances/plugins/connections/__init__.py`)
- **Smart Plugin** (`glances/plugins/smart/__init__.py`)
- **Sensors Plugin** (`glances/plugins/sensors/__init__.py`)
- **RAID Plugin** (`glances/plugins/raid/__init__.py`)
- **WiFi Plugin** (`glances/plugins/wifi/__init__.py`)
- **Memory Swap Plugin** (`glances/plugins/memswap/__init__.py`)
- **Core Plugin** (`glances/plugins/core/__init__.py`)
- **Quicklook Plugin** (`glances/plugins/quicklook/__init__.py`)
- **Uptime Plugin** (`glances/plugins/uptime/__init__.py`)

These plugins check for `self.input_method == 'snmp'` and adjust their behavior accordingly, often by disabling functionality not available via SNMP or implementing alternative data collection methods.

## Application Integration

### Command-Line Arguments (`glances/main.py`)

SNMP-related command-line options:
```bash
--snmp-community SNMP_COMMUNITY   # SNMP community (default: public)
--snmp-port SNMP_PORT             # SNMP port (default: 161)
--snmp-version SNMP_VERSION       # SNMP version: 1, 2c, 3 (default: 2c)  
--snmp-user SNMP_USER             # SNMP username (SNMPv3 only)
--snmp-auth SNMP_AUTH             # SNMP authentication key (SNMPv3 only)
--snmp-force                      # Force SNMP mode instead of trying Glances server
```

### Client Connection Logic (`glances/client.py`)

**Connection Flow**:
1. **Default**: Try to connect to Glances server first
2. **Fallback**: Automatically fall back to SNMP if Glances server unavailable
3. **Force**: Use `--snmp-force` to skip Glances server and use SNMP directly

**Key Methods**:
- `_login_snmp()`: Initialize SNMP connection and verify connectivity
- `update_snmp()`: Update statistics using SNMP method
- `client_mode`: Property managing connection type ('glances' or 'snmp')

### Browser Integration (`glances/client_browser.py`)

**Status Indicators**:
- 'SNMP': Connected via SNMP
- 'ONLINE': Connected via native Glances server  
- 'OFFLINE': No connection

## SNMP MIB Standards Used

### Standard MIBs:
- **SNMPv2-MIB**: Basic system information (1.3.6.1.2.1.1.*)
- **IF-MIB**: Network interface statistics (1.3.6.1.2.1.2.*)
- **HOST-RESOURCES-MIB**: Windows system resources (1.3.6.1.2.1.25.*)

### Vendor/Implementation-Specific MIBs:
- **UCD-SNMP-MIB**: NET-SNMP daemon statistics (1.3.6.1.4.1.2021.*)
- **NetApp**: Enterprise-specific OIDs (1.3.6.1.4.1.789.*)

## Configuration and Documentation

### Documentation Files:
- `docs/man/glances.1`: Manual page with SNMP options
- `docs/quickstart.rst`: Quick start guide with SNMP examples
- `docs/cmds.rst`: Command reference
- `README.rst`: Installation and dependency information

### Installation:
- Optional dependency: `pip install pysnmp-lextudio<6.3.1` (version pinned for stability)
- Available via: `pip install glances[snmp]`
- Part of the comprehensive install: `pip install glances[all]`

### Requirements Files:
- Listed in `optional-requirements.txt` as `pysnmp-lextudio<6.3.1`
- Defined in `pyproject.toml` under `[project.optional-dependencies]`

## Limitations and Considerations

### SNMP Version Limitations:
- **v1**: No bulk requests (inefficient for large tables)
- **v2c**: Community-based security (plain text)
- **v3**: User-based security but more complex setup

### Platform Support:
- **Linux**: Full support via NET-SNMP
- **Windows**: Uses HOST-RESOURCES-MIB  
- **VMware ESXi**: Special handling for memory stats
- **NetApp**: Enterprise-specific extensions

### Performance Considerations:
- Bulk requests used when possible for efficiency
- OS-specific optimizations based on detected system type
- Automatic fallback mechanisms for unsupported features

## Error Handling

### Connection Issues:
- Automatic fallback from Glances server to SNMP
- Graceful handling of unreachable SNMP agents
- Timeout and retry mechanisms

### Data Collection Issues:
- Empty OID response handling
- Missing MIB support detection
- Plugin-specific error recovery

## Future Improvements

### Potential Enhancements:
1. **SNMPv3 Security**: Enhanced authentication and privacy protocols
2. **Custom MIB Support**: User-defined OID mappings
3. **SNMP Discovery**: Automatic network device detection
4. **Performance Optimization**: Caching and batch requests
5. **Extended Platform Support**: Additional vendor-specific implementations

## Summary

Glances provides comprehensive SNMP support as a monitoring fallback mechanism. The implementation covers:

- **9 core plugins** with SNMP OID definitions
- **3 SNMP protocol versions** (1, 2c, 3)  
- **7 OS/platform types** with specific optimizations
- **Standards-compliant** MIB usage
- **Robust error handling** and fallback mechanisms
- **Flexible configuration** via command-line options

The SNMP functionality enables Glances to monitor remote systems and network devices that don't support the native Glances server, making it a versatile monitoring solution for heterogeneous environments.