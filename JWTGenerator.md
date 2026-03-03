context에 다음의 것들이 필요하다.
기본세팅은 
aud : client id
sub : principal
scope : getAuthorizedScopes()

TADA의 경우, 
client_id : 'tada_rider_app',
scope: read, write, trust 
user_name: <- custom claim!  , 계정 uuid
exp
jti
authorities : `["ROLE_INSECURE_USER"]`
![[Pasted image 20240221161324.png]]


