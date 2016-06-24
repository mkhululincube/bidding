<?php
class ModelPaymentPaynow extends Model {
    public function getMethod($address, $total) {
            $this->load->language('payment/paynow');

            $this->load->model('setting/setting');

            $setting = $this->model_setting_setting->getSetting('paynow', (int)$this->config->get('paynow_config_store_id'));
            if($setting['paynow_config_merchant_id'] &&  $setting['paynow_status'])
            {
                $status =  true;
            }
            else
            {
                $status =  false;
            }
            
            $method_data = array();

            if ($status) {
            $method_data = array(
                    'code'       => 'paynow',
                    'title'      => $this->language->get('text_title'),
                    'sort_order' => $setting['paynow_sort_order']
            );
        }
        return $method_data;
    }

    public function update($order_id, $browser_url, $poll_url, $paynow_reference, $amount, $status ) {
		$query = $this->db->query("SELECT * FROM `" . DB_PREFIX . "paynow` WHERE `order_id` = '" . (int)$order_id . "'");
        if ($query->num_rows) {        	
                $this->db->query("UPDATE `" . DB_PREFIX . "paynow` SET "
                    . " `browser_url` = '" . $browser_url . "',"
                    . " `poll_url` = '" . $poll_url . "',"
                    . " `paynow_reference` = '" . $paynow_reference . "',"
                    . " `amount` = '" . $amount . "',"
                    . " `status` = '" . $status . "'"
                    . " WHERE `order_id` = '" . (int)$order_id . "'"
                    );
        } else {								
                $this->db->query("INSERT INTO `" . DB_PREFIX . "paynow` (`order_id`, `browser_url`, `poll_url`, `paynow_reference`, `amount`, `status`) "
                    . " VALUES ('" . (int)$order_id . "',"
                    . " '" . $browser_url . "',"
                    . " '" . $poll_url . "',"
                    . " '" . $paynow_reference . "',"
                    . " '" . $amount . "',"
                    . " '" . $status . "')"
                    );
        }
    }

    public function getPaynowInfo($order_id) {
        $query = $this->db->query("SELECT * FROM `" . DB_PREFIX . "paynow` WHERE `order_id` = '" . (int)$order_id . "'");
        return $query->row;
    }

    public function getOpenPaynow() {
        $query = $this->db->query("SELECT `order_id` FROM `" . DB_PREFIX . "paynow` WHERE `status` = 'Sent to Paynow'");
        return $query->rows;
    }

    public function getMerchantKey($store_id){
            $query = $this->db->query("SELECT * FROM `" . DB_PREFIX . "setting` WHERE `store_id` = '" . (int)$store_id . "' AND `key` = 'paynow_config_merchant_key'");

            $row = $query->row;
            return $row['value'];
    }
	
    public function getTransactionFinalStatus($store_id){
            $query = $this->db->query("SELECT * FROM `" . DB_PREFIX . "setting` WHERE `store_id` = '" . (int)$store_id . "' AND `key` = 'paynow_default_success_status'");

            $row = $query->row;
            return $row['value'];
    }
}
?>
