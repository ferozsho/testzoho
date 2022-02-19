# Zoho Plugin!

This integration, which is in the form of a plugin, allows you to easily embed your Contact pages and payment Lead or Deal in your WordPress site.


# Installation

 1. Install the zoho plugin pro
 2. Under settings tab add the {License Key}
 3. Zoho Accounts
	 1. Add New Account
	 2. Account Name (Account #1)
	 3. Click on Login with Zoho Button
	 4. Authorization with CRM administrator privilege
	 5. Test Connection

## Payment (Lead or Deal)

 - Upon payment Failed, Change to LEAD
 - Upon payment Success, Change to DEAL

> File Changes:

|Mode                |Description
|---------------|-------------------------------|
|File Name		|`functions.php`            |
|Path          	|`wp-content/themes/videotube-child/`          |

**Line Number :** 4
**Action:** Add
**CODE:**

    if(class_exists('vxcf_zoho')){
    	require_once(WP_PLUGIN_DIR.'/cf7-zoho-pro/cf7-zoho-pro.php');
    }

**Line Number :** 5932
**Action:** Add
**CODE:**

    add_action('woocommerce_after_order_object_save', 'create_lead_deal_in_zoho_crm', 15, 2);

    function create_lead_deal_in_zoho_crm($order_id, $order){
      $order = wc_get_order($order_id);
      $orderData = $order->get_data();
      $firstName = isset($orderData['billing']) && strlen($orderData['billing']['first_name']) > 0 ? $orderData['billing']['first_name'] : "";
      $lastName = isset($orderData['billing']['last_name']) && strlen($orderData['billing']['last_name']) > 0 ? $orderData['billing']['last_name'] : "";
      $phoneNumber = isset($orderData['billing']['phone']) && strlen($orderData['billing']['phone']) > 0 ? $orderData['billing']['phone'] : "";
      $email = isset($orderData['billing']['email']) && strlen($orderData['billing']['email']) > 0 ? $orderData['billing']['email'] : "";
      $city = isset($orderData['billing']['city']) && strlen($orderData['billing']['city']) > 0 ? $orderData['billing']['city'] : "";
      $state = isset($orderData['billing']['state']) && strlen($orderData['billing']['state']) > 0 ? $orderData['billing']['state'] : "";
      $country = isset($orderData['billing']['country']) && strlen($orderData['billing']['country']) > 0 ? $orderData['billing']['country'] : "";
      $zipCode = isset($orderData['billing']['postcode']) && strlen($orderData['billing']['postcode']) > 0 ? $orderData['billing']['postcode'] : "";
      $street = isset($orderData['billing']['address_1']) && strlen($orderData['billing']['address_1']) > 0 ? $orderData['billing']['address_1'] : "";

      $isMultipleCourseOpted = false;
      $orderItems = [];
      foreach($order->get_items() as $item_id => $item){
            $orderItems[] = $item->get_name();
      }
      if(count($orderItems) > 1) $isMultipleCourseOpted = true;
      $items = implode(',', $orderItems);
      $clientIp = isset($_SERVER["HTTP_CF_CONNECTING_IP"]) && $_SERVER["HTTP_CF_CONNECTING_IP"] != "" ? $_SERVER["HTTP_CF_CONNECTING_IP"] : $_SERVER['REMOTE_ADDR'];

      if( in_array( $order->get_status(), ['processing', 'successful'] ) ) {
          $dealName = $firstName.$lastName;
          $cf7Zoho = new vxcf_zoho;
          $info = $cf7Zoho->get_info(2);
          $api = $cf7Zoho->get_api($info);
          $tokenInformations = $api->get_token();
          $accessToken = $tokenInformations['access_token'];
        if($isMultipleCourseOpted) {
          $courseOptedKey = 'Courses_Opted';
          $items = $orderItems;
        }else{
          $courseOptedKey = 'Course_Opted';
        }
        $payment_method_string = '';
        $payment_method = $order->get_payment_method();
        if ( $payment_method ) {
          /* translators: %s: payment method */
          $payment_method_string = sprintf(
            __( 'Payment via %s', 'woocommerce' ),
            esc_html( isset( $payment_gateways[ $payment_method ] ) ? $payment_gateways[ $payment_method ]->get_title() : $payment_method )
          );

          if ( $transaction_id = $order->get_transaction_id() ) {
            if ( isset( $payment_gateways[ $payment_method ] ) && ( $url = $payment_gateways[ $payment_method ]->get_transaction_url( $order ) ) ) {
              $payment_method_string .= ' (<a href="' . esc_url( $url ) . '" target="_blank">' . esc_html( $transaction_id ) . '</a>)';
            } else {
              $payment_method_string .= ' (' . esc_html( $transaction_id ) . ')';
            }
          }
        }

        $jsonPostField = [
          'data' => [
            [
              'Deal_Name' => $dealName,
              'Stage' => "Closed Won",
              'Pipeline' => "Standard (Standard)",
              'Mobile' => $phoneNumber,
              'Email' => $email,
              'Phone' => $phoneNumber,
              'Lead_Source' => "Online Store",
              $courseOptedKey => $items,
              'Street' => $street,
              'City' => $city,
              'State' => $state,
              'Country' => $country,
              'Zip_Code' => $zipCode,
              'Amount' => $order->get_total(),
              'Customer_IP' => $clientIp,
              'Lead_Type' => "ESales",
              'Description' => "Order #" . $order->get_order_number() . $payment_method_string,
            ]
          ],
          'duplicate_check_fields' => [
            'Email',
            'Mobile'
          ],
          'trigger' => [
            'workflow'
          ]
        ];
        callZohoApi(json_encode($jsonPostField,true), $accessToken, 'Deals');
      } else if( in_array( $order->get_status(), ['failed'] ) ) {
        $cf7Zoho = new vxcf_zoho;
        $info = $cf7Zoho->get_info(2);
        $api = $cf7Zoho->get_api($info);
        $tokenInformations = $api->get_token();
        $accessToken = $tokenInformations['access_token'];
        if($isMultipleCourseOpted) {
          $courseOptedKey = 'Courses_Opted';
          $items = $orderItems;
        }else{
          $courseOptedKey = 'Course_Opted';
        }
        $jsonPostField = [
        'data' => [
          [
          'First_Name' => $firstName,
          'Last_Name' => $firstName.' '.$lastName,
          'Mobile' => $phoneNumber,
          $courseOptedKey => $items,
          'Lead_Source' => "Online Store",
          'Email' =>  $email,
          'City' =>  $city,
          'State' => $state,
          'Country' => $country,
          'Zip_Code' => $zipCode,
          'Customer_IP' => $clientIp,
          'Location_Link_GMaps' => "https://www.medvarsity.com/ip_address.php?IP=$clientIp",
          'Created_Via' => "https://test.medvarsity.com",
          'Description' => "Order Created in website and payment may failed so creating as lead",
          'Lead_Type' => "ESales"
          ]
        ],
        'duplicate_check_fields' => [
          'Email',
          'Mobile'
        ],
        'trigger' => [
          'workflow'
        ]
        ];
        callZohoApi(json_encode($jsonPostField,true), $accessToken, 'Leads');
      }
    }        

    function callZohoApi($json, $accessToken, $module) {
      $curl = curl_init();
      curl_setopt_array($curl, array(
       CURLOPT_URL => 'https://www.zohoapis.com/crm/v2/'.$module.'/upsert',
       CURLOPT_RETURNTRANSFER => true,
       CURLOPT_ENCODING => '',
       CURLOPT_MAXREDIRS => 10,
       CURLOPT_TIMEOUT => 0,
       CURLOPT_FOLLOWLOCATION => true,
       CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
       CURLOPT_CUSTOMREQUEST => 'POST',
       CURLOPT_POSTFIELDS => $json,
         CURLOPT_HTTPHEADER => array(
            'Authorization: Bearer '.$accessToken,
            'Content-Type: application/json'
         ),
      ));

      $response = curl_exec($curl);

      curl_close($curl);
      echo $response;
    }
___

> File Changes:

|Mode                |Description
|---------------|-------------------------------|
|File Name		|`cf7-zoho-pro.php`            |
|Path          	|`wp-content/plugins/cf7-zoho-pro/`          |

**Line Number :** 62
**Action:** Add
**CODE:**

   	public static $geo_location_details;

**Line Number :** 98
**Action:** Add 
**CODE:**
#### function setup_main()  
    $curl = curl_init();
    $clientIp = isset($_SERVER["HTTP_CF_CONNECTING_IP"]) && $_SERVER["HTTP_CF_CONNECTING_IP"] != "" ? $_SERVER["HTTP_CF_CONNECTING_IP"] : $_SERVER['REMOTE_ADDR'];
    $token = base64_encode(GEO_LOCATION_API_CLIENT_ID.':'.GEO_LOCATION_API_PASSWORD);
    curl_setopt_array($curl, array(
      CURLOPT_URL => GEO_LOCATION_API_URL.$clientIp.'?pretty',
      CURLOPT_RETURNTRANSFER => true,
      CURLOPT_ENCODING => '',
      CURLOPT_MAXREDIRS => 10,
      CURLOPT_TIMEOUT => 0,
      CURLOPT_FOLLOWLOCATION => true,
      CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
      CURLOPT_CUSTOMREQUEST => 'GET',
      CURLOPT_HTTPHEADER => array(
      'Authorization: Basic '.$token
      ),
    ));

    $response = curl_exec($curl);
    curl_close($curl);
    self::$geo_location_details = json_decode($response,true);

**Line Number :** 170
**Action:** Add
**CODE:**
#### form_submitted() function
    $userLocation = self::$geo_location_details;
    $city = $country = $state = $zipCode = "";	
    if (!empty($userLocation) && is_array($userLocation) && count($userLocation) > 0) {
      $city = isset($userLocation['city']) && isset($userLocation['city']['names']) ? $userLocation['city']['names']['en'] : "";
      $state = isset($userLocation['subdivisions']) ? $userLocation['subdivisions'][0]['names']['en'] : "";
      $country = isset($userLocation['country']) ? $userLocation['country']['names']['en'] : "";
      $zipCode = isset($userLocation['postal']) ? $userLocation['postal']['code'] : "";
    }	
    $locationData = ['City' => $city, 'State' => $state, 'Country' => $country, 'Zip_Code' => $zipCode];

**Line Number :** 175
**Action:** Add
**CODE:**
#### form_submitted() function

    if(in_array($name, ["City", "State", "Country", "Zip_Code"])) {
	    $val = $locationData[$name];
    }




