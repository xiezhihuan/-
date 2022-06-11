# oslo.config

oslo.config是一个读取配置参数的pyhton库.

下面是简单是使用示例：

```python
from oslo_config import cfg

CONF = cfg.CONF

opts = [
    cfg.HostAddressOpt('host_ip', 
                        default = '127.0.0.1', 
                        help = 'The IP address on which cyborg-api listens.'),
]

opt_group = cfg.OptGroup(name='api',
                         title='Options for the cyborg-api service')


```
## 

Type | Option
---  | :---
oslo_config.types.String | oslo_config.cfg.StrOpt
oslo_config.types.String|       oslo_config.cfg.SubCommandOpt|
oslo_config.types.Boolean|      oslo_config.cfg.BoolOpt|
oslo_config.types.Integer|      oslo_config.cfg.IntOpt|
oslo_config.types.Float|        oslo_config.cfg.FloatOpt|
oslo_config.types.Port|         oslo_config.cfg.PortOpt|
oslo_config.types.List|         oslo_config.cfg.ListOpt|
oslo_config.types.Dict|         oslo_config.cfg.DictOpt|
oslo_config.types.IPAddress|    oslo_config.cfg.IPOpt|
oslo_config.types.Hostname|     oslo_config.cfg.HostnameOpt|
oslo_config.types.HostAddress|  oslo_config.cfg.HostAddressOpt|
oslo_config.types.URI|          oslo_config.cfg.URIOpt|