<?php 
class ControllerPaymentPaynow extends Controller {
	public $final_order_status = 15;
	public $paynow_initiate_transaction_url = 'https://www.paynow.co.zw/Interface/InitiateTransaction';
	
	private function setClassData() { 
		$this->load->model('setting/setting');
		$store_info = $this->model_setting_setting->getSetting('paynow');
		
		//override if set
		if (array_key_exists('paynow_default_success_status', $store_info))
		{
			$this->final_order_status = $store_info['paynow_default_success_status'];
		}
		
		if (array_key_exists('paynow_initiate_transaction_url', $store_info))
		{
			$this->paynow_initiate_transaction_url = $store_info['paynow_initiate_transaction_url'];
		}
	}
	
	public function index() { 
         	$data['button_confirm'] = $this->language->get('button_confirm');

		$data['continue'] = $this->url->link('checkout/success');

	
			$this->load->language('payment/paynow');
            $this->load->model('checkout/order');

            $order_id = $this->session->data['order_id'];
            $order_info = $this->model_checkout_order->getOrder($order_id);
			if ($order_info) {
				//set customer email
				$custEmail = "";
				if (isset($this->session->data['guest'])) { 
					$custEmail = $this->session->data['guest']['email'];  // Guest user's email
				} elseif($this->customer->isLogged()) {
					$custEmail = $this->customer->getEmail(); // Customer's email
				} 

                $MerchantId =       $this->config->get('paynow_config_merchant_id');
                $MerchantKey =      $this->config->get('paynow_config_merchant_key');
                $ConfirmUrl =       str_replace('&amp;', '&', $this->url->link('payment/paynow/callback', 'order_id=' . $order_id));
                $ReturnUrl =        str_replace('&amp;', '&', $this->url->link('payment/paynow/callback', 'order_id=' . $order_id));
                $Reference =        $this->session->data['order_id'];
                $Amount =           $order_info["total"];
                $AdditionalInfo =   "";
                $Status =           "Message";

                //set POST variables
                $values = array('resulturl' => $ConfirmUrl,
                            'returnurl' => $ReturnUrl,
                            'reference' => $Reference,
                            'amount' => $Amount,
                            'id' => $MerchantId,
                            'additionalinfo' => $AdditionalInfo,
							'authemail' => $custEmail,
                            'status' => $Status);
				
                $fields_string = $this->CreateMsg($values, $MerchantKey);
 				
                //open connection
                $ch = curl_init();
	
				$this->setClassData();
                //set the url, number of POST vars, POST data
                $url = $this->paynow_initiate_transaction_url;				
                curl_setopt($ch, CURLOPT_URL, $url);
                curl_setopt($ch, CURLOPT_POST, true);
                curl_setopt($ch, CURLOPT_POSTFIELDS, $fields_string);
                curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);

                //execute post
                $result = curl_exec($ch);
				//$result = curl_getinfo($ch);
                if($result) {
					
					//close connection
                    $msg = $this->ParseMsg($result);

					//first check status, take appropriate action
                    if (strtolower($msg["status"]) == $this->language->get('ps_error')){
                        $error = "Failed with error: " . $msg["Error"];
                    }
                    else if (strtolower($msg["status"]) == $this->language->get('ps_ok')){
						
						//second, check hash
                        $validateHash = $this->CreateHash($msg, $MerchantKey);
                        if($validateHash != $msg["hash"]){
                            $error =  "Paynow reply hashes do not match : " . $validateHash . " - " . $msg["hash"];
                        }
                        else
                        {
                            $theProcessUrl = $msg["browserurl"];

                            $this->load->model('payment/paynow');		
							
																					
                            $this->model_payment_paynow->update($order_id
                                    , $msg["browserurl"]
                                    , $msg["pollurl"]
                                    , 0
                                    , ""
                                    , $order_info["total"]
                                    , "Sent to Paynow"
                                    );
                        }
                    }
                    else {						
						//unknown status
						$error =  "Invalid status in from Paynow, cannot continue.";
                    }

                }
                else
                {
                   $error = curl_error($ch);
                }

                curl_close($ch);
            }

            if(isset($error))
            {			
                $data['error'] = $error;

                if (file_exists(DIR_TEMPLATE . $this->config->get('config_template') . '/template/payment/paynow_failed.tpl')) {
                    $this->template = $this->config->get('config_template') . '/template/payment/paynow_failed.tpl';
                } else {
                    $this->template = 'default/template/payment/paynow_failed.tpl';
                }
            }
            else
            { 
                $data['action'] = $theProcessUrl;
              
                $bits = explode ("=", $theProcessUrl);
				
				if (array_key_exists(1, $bits) == true)
				{
					$data['authemail'] = $bits[1];		
							
				}
                
                	if (file_exists(DIR_TEMPLATE . $this->config->get('config_template') . '/template/payment/paynow.tpl')) {
						return $this->load->view($this->config->get('config_template') . '/template/payment/paynow.tpl', $data);
					} else {
						return $this->load->view('default/template/payment/paynow.tpl', $data);
					}
		
            }

         
        }

	public function confirm() {
	
			//file_put_contents('phperrorlog.txt','CONFIRM THIS: '.print_r($_POST, true), FILE_APPEND | LOCK_EX);
            if($this->request->server['REQUEST_METHOD'] == 'POST')
            {
                $order_id = $this->request->post['order_id'];
				
				$this->load->language('payment/paynow');

                $this->load->model('payment/paynow');
                $paynowInfo = $this->model_payment_paynow->getPaynowInfo($order_id);

                if($paynowInfo)
                {
                	
					//close connection
                    $msg = $this->request->post;

                    $MerchantKey =  $this->config->get('paynow_config_merchant_key');
                    $validateHash = $this->CreateHash($msg, $MerchantKey);
                    if($validateHash != $msg["hash"]){
                        $data['text_message'] =  "Paynow reply hashes do not match : " . $validateHash . " - " . $msg["hash"];
                    }
                    else
                    {
                        $this->model_payment_paynow->update($order_id
                                    , $paynowInfo['resulturl']
                                    , $msg["pollurl"]
                                    , $msg["paynowreference"]
                                    , $msg["amount"]
                                    , $msg["status"]
                                    );
                        $this->load->model('checkout/order');

                        if (trim(strtolower($msg["status"])) == $this->language->get('ps_created_but_not_paid')){
                            //$data['text_message'] = "Transaction has not yet been paid on Paynow.<br /><br />"
                            //       . "<a class='s_button_1 s_main_color_bgr' href='" . $paynowInfo['process_url'] . "'><span class='s_text'>Return to Paynow</span></a>";
                        }
                        else if (trim(strtolower($msg["status"])) == $this->language->get('ps_cancelled')){
                            //$data['text_message'] = "Transaction was cancelled by the user.  You will be redirected back to the checkout page.<br /><br />";
                            //$data['redirect_url'] = str_replace('&amp;', '&', $this->url->link('checkout/checkout'));
                            $order = $this->model_checkout_order->getOrder($order_id);
                            if($order["order_status_id"] != 7){
                                $this->model_checkout_order->confirm($order_id, 7, "Cancelled by user.", true);
                            }
                        }
                        else if (trim(strtolower($msg["status"])) == $this->language->get('ps_failed')){
                            //$data['text_message'] = "Transaction payment has failed on Paynow.   You can still resume the payment.<br /><br />"
                            //         . "<a class='s_button_1 s_main_color_bgr' href='" . $paynowInfo['process_url'] . "'><span class='s_text'>Return to Paynow</span></a>";
                        }
                        else if (trim(strtolower($msg["status"])) == $this->language->get('ps_paid') || trim(strtolower($msg["status"])) == $this->language->get('ps_awaiting_delivery') || trim(strtolower($msg["status"])) == $this->language->get('ps_delivered')){
                            //$data['text_message'] = "Transaction Paid Successfully.  You will be redirected to a confirmation page.<br /><br />";
                            //$data['redirect_url'] = str_replace('&amp;', '&', $this->url->link('checkout/success'));
                            $order = $this->model_checkout_order->getOrder($order_id);
							$this->setClassData();

                            if($order["order_status_id"] != $this->final_order_status){
                                $this->model_checkout_order->confirm($order_id, $this->final_order_status, "Paynow ID: " . $msg["PaynowReference"], true);
                            }
                        }
                        else {
                           //$data['text_message'] = "Invalid status in from Paynow, cannot continue.";
                        }
                    }
                }
                else
                {
                   //$data['text_message'] = "Could not update order info from Paynow.";
                }
            }
            else
            {
                //$data['text_message'] =  "paynow order info not found.";
            }
	}

	public function callback() {
		//file_put_contents('phperrorlog.txt','CALLBACK THIS: '.print_r($_POST, true), FILE_APPEND | LOCK_EX);
		$order_id = $this->request->get['order_id'];
		$data['continue'] = $this->url->link('checkout/success');
		
		$this->load->language('payment/paynow');
		$this->load->model('payment/paynow');
		
		$paynowInfo = $this->model_payment_paynow->getPaynowInfo($order_id);
		$data['redirect_url'] = "";
		
		if ($paynowInfo)
		{
			$this->session->data['order_id'] = $order_id;

			$ch = curl_init();

			//set the url, number of POST vars, POST data
			curl_setopt($ch, CURLOPT_URL, $paynowInfo['poll_url']);
			curl_setopt($ch, CURLOPT_POST, 0);
			curl_setopt($ch, CURLOPT_POSTFIELDS, '');
			curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
			curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);

			//execute post
			$result = curl_exec($ch);
 
			if($result) {

				//close connection
				$msg = $this->ParseMsg($result);
				
				$MerchantKey =  $this->config->get('paynow_config_merchant_key');
				$validateHash = $this->CreateHash($msg, $MerchantKey);
  
				if($validateHash != $msg["hash"]){
					$data['text_message'] =  "Paynow reply hashes do not match : " . $validateHash . " - " . $msg["hash"];
				}
				else
				{
					$this->model_payment_paynow->update($order_id
								, $paynowInfo['browser_url']
								, $msg["pollurl"]
								, $msg["paynowreference"]
								, $msg["amount"]
								, $msg["status"]
								);
					$this->load->model('checkout/order');

					if (trim(strtolower($msg["status"])) == $this->language->get('ps_awaiting_redirect') || trim(strtolower($msg["status"])) == $this->language->get('ps_created_but_not_paid')){
						$data['text_message'] = "Transaction has not yet been paid on Paynow.<br /><br />"
								 . "<a class='s_button_1 s_main_color_bgr' href='" . $paynowInfo['browser_url'] . "'><span class='s_text'>Return to Paynow</span></a>";
					}
					else if (trim(strtolower($msg["status"])) == $this->language->get('ps_cancelled')){

						$data['text_message'] = "Transaction was cancelled by the user.  You will be redirected back to the checkout page.<br /><br />";
						$data['redirect_url'] = str_replace('&amp;', '&', $this->url->link('checkout/checkout'));

						$order = $this->model_checkout_order->getOrder($order_id);
						if($order["order_status_id"] != 7){
								
							$data['text_message'] = "Transaction was cancelled by the user.  You will be redirected back to the checkout page.<br /><br />";
                            $data['redirect_url'] = str_replace('&amp;', '&', $this->url->link('checkout/checkout'));
							$this->model_checkout_order->addOrderHistory($order_id, 7, "Cancelled by user.", true);
							//$this->model_checkout_order->confirm($order_id, 7, "Cancelled by user.", true);
						}
					}
					else if (trim(strtolower($msg["status"])) == $this->language->get('ps_failed')){
						$data['text_message'] = "Transaction payment has failed on Paynow.   You can still resume the payment.<br /><br />"
								 . "<a class='s_button_1 s_main_color_bgr' href='" . $paynowInfo['browser_url'] . "'><span class='s_text'>Return to Paynow</span></a>";
					}
					else if (trim(strtolower($msg["status"])) == $this->language->get('ps_paid') || trim(strtolower($msg["status"])) == $this->language->get('ps_awaiting_delivery') || trim(strtolower($msg["status"])) == $this->language->get('ps_delivered')){
						$data['text_message'] = "Transaction Paid Successfully.  You will be redirected to a confirmation page.<br /><br />";
						$data['redirect_url'] = str_replace('&amp;', '&', $this->url->link('checkout/success'));
						

						$order = $this->model_checkout_order->getOrder($order_id);
						if($order["order_status_id"] != $this->final_order_status){
							
							$this->model_checkout_order->addOrderHistory($order_id, $this->final_order_status);
							
							
						}
					}
					else {
					   $data['text_message'] = "Invalid status in from Paynow, cannot continue.";
					}
				}
			}
			else
			{
			    $data['text_message'] = "Could not update order info from Paynow.";
			    //TODO do we trust this?  Probably not... should still get the results to confirm.
					
				//trigger_error(sprintf('Curl failed with error #%d: %s',	$e->getCode(), $e->getMessage()),E_USER_NOTICE);
			}

			curl_close($ch);
		}
		else
		{
			$data['text_message'] =  "Paynow order info not found.";
		}
		$data['heading_title'] = "Paynow Results";

		$this->response->redirect($data['redirect_url']);
		
	}

	//Has not been updated to Paynow
	public function dayend() {
		$key = $this->request->get['key'];
		if($key != "Paynowayend") { echo "Invalid Key"; die(); }

		$this->load->language('payment/paynow');
		$this->load->model('checkout/order');
		$this->load->model('payment/paynow');
		$order_ids = $this->model_payment_paynow->getOpenPaynow();

		foreach ($order_ids as $id) {
			echo "Processing Order: " . $order_id . "<br />";
			$order_id = $id['order_id'];

			$order = $this->model_checkout_order->getOrder($order_id);
			$paynowInfo = $this->model_payment_paynow->getVpaymentInfo($order_id);

			if($paynowInfo)
			{
				$ch = curl_init();

				//set the url, number of POST vars, POST data
				curl_setopt($ch, CURLOPT_URL, $paynowInfo['check_url']);
				curl_setopt($ch, CURLOPT_POST, 0);
				curl_setopt($ch, CURLOPT_POSTFIELDS, '');
				curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);

				//execute post
				$result = curl_exec($ch);

				if($result) {
					//close connection
					$msg = $this->ParseMsg($result);

					$MerchantKey =  $this->model_payment_paynow->getMerchantKey($order["store_id"]);
					$validateHash = $this->CreateHash($msg, $MerchantKey);

					if($validateHash != $msg["hash"]){
						echo "Paynow reply hashes do not match : " . $validateHash . " - " . $msg["hash"] ."<br /><br />";
					}
					else
					{
						$this->model_payment_paynow->update($order_id
									, $paynowInfo['process_url']
									, $msg["checkurl"]
									, $msg["reference"]
									, $msg["bankreference"]
									, $msg["amount"]
									, $msg["status"]
									);

						if (trim(strtolower($msg["status"])) == $this->language->get('ps_awaiting_redirect') || trim(strtolower($msg["status"])) == $this->language->get('ps_created_but_not_paid')){
								echo "Transaction has not yet been paid on Paynow.<br /><br />";
						}
						else if (trim(strtolower($msg["status"])) == $this->language->get('ps_cancelled')){

							echo "Transaction was cancelled by the user.  You will be redirected back to the checkout page.<br /><br />";

							$order = $this->model_checkout_order->getOrder($order_id);
							if($order["order_status_id"] != 7){
								$this->model_checkout_order->confirm($order_id, 7, "Cancelled by user.", true);
							}
						}
						else if (trim(strtolower($msg["status"])) == $this->language->get('ps_failed')){
							echo "Transaction payment has failed on Paynow.   You can still resume the payment.<br /><br />";
						}
						else if (trim(strtolower($msg["status"])) == $this->language->get('ps_paid') || trim(strtolower($msg["status"])) == $this->language->get('ps_awaiting_delivery')){
							echo "Transaction Paid Successfully.  You will be redirected to a confirmation page.<br /><br />";
							$this->setClassData();

							$order = $this->model_checkout_order->getOrder($order_id);
							if($order["order_status_id"] != $this->final_order_status){
								$this->model_checkout_order->confirm($order_id, $this->final_order_status, "Paynow ID: " . $msg["PaynowReference"], true);
							}
						}
						else {
						   echo "Invalid status in from Paynow, cannot continue.<br /><br />";
						}
					}
				}
				else
				{
					echo "Could not update order info from Paynow.<br /><br />";
					//TODO do we trust this? Probably not... should still get the results to confirm.
					
					//trigger_error(sprintf('Curl failed with error #%d: %s',	$e->getCode(), $e->getMessage()),E_USER_NOTICE);
				}

				curl_close($ch);
			}
			else
			{
				echo "Paynow order info not found.<br /><br />";
			}
		}
		echo "Dayend checks done.";
	}

	private function ParseMsg($msg) {
		//convert to array data
		$parts = explode("&",$msg);
		$result = array();
		foreach($parts as $i => $value) {
			$bits = explode("=", $value, 2);
			$result[$bits[0]] = urldecode($bits[1]);
		}

		return $result;
	}

	private function UrlIfy($fields) {
		//url-ify the data for the POST
		$delim = "";
		$fields_string = "";
		foreach($fields as $key=>$value) {
			$fields_string .= $delim . $key . '=' . $value;
			$delim = "&";
		}

		return $fields_string;
	}

	private function CreateHash($values, $MerchantKey){
		$string = "";
		foreach($values as $key=>$value) {
			if( strtoupper($key) != "HASH" ){
				$string .= $value;
			}
		}
		$string .= $MerchantKey;
		$hash = hash("sha512", $string);
		return strtoupper($hash);
	}

	private function CreateMsg($values, $MerchantKey){
		$fields = array();
		foreach($values as $key=>$value) {
		   $fields[$key] = urlencode($value);
		}

		$fields["hash"] = urlencode($this->CreateHash($values, $MerchantKey));

		$fields_string = $this->UrlIfy($fields);
		return $fields_string;
	}
}
?>
