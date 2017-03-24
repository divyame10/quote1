<?php /** @var $this Ophirah_Qquoteadv_Block_Qquote */ ?>
<?php $totalqty = 0; ?>
<?php if ($this->getQuote()->getData() == array()): ?>
<?php $gg =  Mage::helper('checkout/cart')->getQuote()->getSubtotal(); 
   $gg1  = ($gg * 14)/100;
   ?>
<div class="page-title">
   <h1><?php echo $this->__('Request for Quote') ?></h1>
</div>
<div class="cart-empty">
   <p><?php echo $this->__('No Items to display.'); ?></p>
   <p><?php echo $this->__('Click <a href="%s">here</a> to continue shopping.', $this->getUrl()) ?></p>
</div>
<?php else: ?>
<?php $productNames = $this->getNotSalableProductNames($this->getQuote()); ?>
<div class="page-title" style="height:55px;">
   <h3><?php echo $this->__('Request for Quote') ?></h3>
   <?php if ($this->displayAssignedTo()): ?>
   <div class="assigned-to">
      <?php if (Mage::helper('qquoteadv')->getAdminUser() !== null): ?>
      <?php if ($this->isAssignPreviousEnabled()): ?>
      <?php if ($this->helper('customer')->isLoggedIn()): ?>
      <?php
         $customerData = Mage::getSingleton('customer/session')->getCustomer();
         // echo $this->__('Quote will be assigned to <strong>%s</strong>', Mage::helper('qquoteadv')->getAdminUser(Mage::helper('qquoteadv')->getPreviousAssignedAdmin(null, $customerData->getId()))->getName()); ?>
      <?php else: ?>
      <?php //echo $this->__('If the end user has no previous assigned sales representative the quote will be assigned to <strong>%s</strong>.', $this->getAdminUser()->getName()); ?>
      <?php endif; ?>
      <?php else: ?>
      <?php //echo $this->__('Quote will be assigned to <strong>%s</strong>', $this->getAdminUser()->getName()); ?>
      <?php endif; ?>
      <?php endif; ?>
   </div>
   <?php endif; ?>
</div>
<?php echo $this->getMessagesBlock()->getGroupedHtml() ?>
<div class="getquote">
   Note: If appeared Unit price is Zero or if you are not satisfied with the given price, then please click on SUBMIT QUOTE REQUEST button to get best offer from our experienced sales team.
</div>
<form method='post' id="quotelist" name="quotelist"
   action='<?php echo $this->getUrl('qquoteadv/index/quoteRequest', array('_secure' => true)) ?>'>
   <?php if ($this->displayAssignedTo() && $this->getAdminUser() !== null): ?>
   <input type="hidden" name="user_id" value="<?php echo $this->getAdminUser()->getId(); ?>"/>
   <?php endif; ?>
   <table cellspacing="0" border="0" id="shopping-cart-table" class="data-table cart-table">
      <col width="30"/>
      <col width="75"/>
      <col/>
      <!-- col width="10"/ -->
      <col width="250"/>
      <!-- col width="70"/ -->
      <col width="130"/>
      <thead>
         <tr>
            <th class="a-center"><?php echo $this->__('Remove') ?></th>
			<th>Edit</th>
            <th class="a-center">Product Image</th>
            <th><?php echo $this->__('Product Name') ?></th>
            <!-- th><?php //echo $this->__('Edit') ?></th -->
            <th class="a-center"><?php echo $this->__('Product Information') ?></th>
            <th class="a-center"><?php echo $this->__('Unit Price') ?></th>
            <th class="a-center"><?php echo $this->__('Qty') ?> <span class="required">*</span></th>
         </tr>
      </thead>
	  <?php 
	  foreach ($this->getQuote() as $item){
            
                $product = $this->getProduct($item->getProductId());
	  }
				?>
	  
	  
      <tbody>
         <?php
            $i = 0;
            foreach ($this->getQuote() as $item):
            
                $product = $this->getProduct($item->getProductId());
            
                if ($product->getTypeId() == Mage_Catalog_Model_Product_Type::TYPE_CONFIGURABLE) {
                    $childProduct = Mage::getModel('qquoteadv/qqadvproduct')->getConfChildProduct($item->getId());
                } else {
                    $childProduct = $product;
                }
                ?>
         <tr>
            <td class="a-center">
               <input type="hidden" id="quote_id" name="quote_id" value="<?php echo $item->getQuoteId() ?>">
               <input type="hidden" class="input-text" name="quote[<?php echo $item->getId() ?>][product]"
                  value="<?php echo $item->getProductId(); ?>" size="3"/>
               <input type="hidden" class="input-text"
                  name="quote_request[<?php echo $item->getId() ?>][product_id]"
                  value="<?php echo $item->getProductId(); ?>" size="6"/>
               <?php
                  $items_qt =Mage::getSingleton('checkout/session')->getQuote()->getAllItems();
                  
                  foreach($items_qt as $item_qt) {
                  	$slitem = $item_qt->getProductId() == $item->getProductId()? $item_qt->getId() : "";
                  }
                  ?>
               <a title="<?php echo $this->__('Remove item') ?>"
                  href="<?php echo $this->getUrl('checkout/cart/delete', array('id' => $slitem)). "?url=".'qquoteadv/index/delete/id/'.$item->getId(); ?>"><img
                  src="<?php echo $this->getSkinUrl('images/btn_trash.gif') ?>" width="30" height="30"
                  alt="<?php echo $this->__('Remove item') ?>"/></a>
				 
            </td>
			 <td>
               <a href="<?php echo $this->getUrl('checkout/cart/configure', array('id' => $slitem)) ?>" onclick="saveProductComment();"><?php echo $this->__("Edit") ?></a>
               <br/>
            </td>
            <td>
               <a class="product-image" href="<?php echo $product->getProductUrl(); ?>"><img
                  src="<?php echo $this->helper('catalog/image')->init($childProduct, 'thumbnail', $item->getSmallImage())->resize(75); ?>"
                  alt="<?php echo $item->getName(); ?>"/></a>
            </td>
            <td class="attributes-col">
               <h4 class="title">
                  <a href="<?php echo $product->getProductUrl() ?>"onclick="saveProductComment();"><?php echo $this->htmlEscape($product->getName()) ?></a>
                  <?php //echo $this->htmlEscape($product->getName()) ?>
               </h4>
               <?php
                  $quoteByProduct = Mage::helper('qquoteadv')->getQuoteItem($product, $item->getAttribute(), null, $item);
                  
                  $qty = $quoteByProduct->getItemsQty();
                  
                  // Select price with or without tax
                  if (Mage::getStoreConfig('tax/cart_display/price') == 2  ):
                      $_finalPrice = $quoteByProduct->getGrandTotal() / ($qty > 0 ? $qty : 1);
                  else:
                      $_finalPrice = $quoteByProduct->getSubtotal() / ($qty > 0 ? $qty : 1);
                  endif;
                  
                  
                  foreach ($quoteByProduct->getAllItems() as $_item) {
                  if ($_item->getProductId() == $product->getId()) {
                      echo $this->getItemHtml($_item);
                  }
                  }
                  ?>
            </td>
           
            <td>
               <div>
                  <B><?php echo $this->__('Model No:- ') ?></B><?php echo $product->getmodelNo(); ?>
               </div>
               <div>
                  <B><?php echo $this->__('Item Code:- ') ?></B><?php echo $product->getsku(); ?>
               </div>
               <div>
                  <B><?php echo $this->__('Finish/Color:- ') ?></B><?php echo $product->getfinishColor(); ?>
               </div>
               <div>
                  <B><?php echo $this->__('Pcs Per Carton:- ') ?></B><?php echo $product->getpcsPerCarton(); ?>
               </div>
            </td>
            <!--td>
               <?php
                  // Retrieve Customer Request data from session
                  $prod_request = "";
                  $placeHolder = true;
                  if($quoteData = $this->getCustomerSession()->getQuoteData()){
                      // Get Client Request
                      if(isset($quoteData['quote_request'][$item->getId()]['client_request'])){
                          $prod_request = trim($quoteData['quote_request'][$item->getId()]['client_request']);
                          if($prod_request != "" && $prod_request != null){
                              $placeHolder = false;
                          }
                      }
                      // Set Product qty
                      if(isset($quoteData['quote_request'][$item->getId()]['qty']['0'])){
                          $item->setQty($quoteData['quote_request'][$item->getId()]['qty']['0']);
                      }
                  }
                  ?>
               
               <?php if($placeHolder){?>
                   <div id="parent">
                       <textarea name="quote_request[<?php echo $item->getId() ?>][client_request]" rows="10" style="width:98%;position:absolute;"><?php echo $prod_request; ?></textarea>
                       <div id="comment" onclick="$(this).hide(); $('textArea').focus();">
                           <?php echo $this->__('Be advised to enter your comments') ?>
                       </div>
                   </div>
               <?php }else{ ?>
                       <textarea name="quote_request[<?php echo $item->getId() ?>][client_request]" rows="4" style="width:98%"><?php echo $prod_request; ?></textarea>
               <?php } ?>
               </td>-->
            <td>
               <?php  if (Mage::helper('qquoteadv/not2order')->getShowPrice($product)): ?>
               <?php echo Mage::helper('checkout')->formatPrice($_finalPrice) ?>
               <?php endif;  ?>
            </td>
            <td nowrap="nowrap">
               <div id="qdiv_<?php echo $item->getId() ?>" nowrap="nowrap">
                  <!-- readonly is removed to make qty editable-->
                  <input type="text" 
                     class="required-entry validate-zero-or-greater required-entry input-text loading"
                     size="6" readonly
                     name="quote_request[<?php echo $item->getId() ?>][qty][]"
                     value="<?php echo $item->getQty(); ?>"/>
                  <?php $totalqty += $item->getQty(); ?>
               </div>
               <div>
                  <?php if ($this->getShowTrierOption()) : ?>
                  <!-- a class="pad" href="#"
                     onClick="addNewLine(<?php echo $item->getId() ?>, 'quote_request[<?php echo $item->getId() ?>][qty][]'); return false;"><?php echo $this->__('Add Qty') ?></a -->
                  <?php endif; ?>
               </div>
               <input type="hidden" name="quote[<?php echo $item->getId() ?>][qty]"
                  value="<?php echo $item->getQty(); ?>" size="3"/>&nbsp;
               <input type="hidden" name="quote[<?php echo $item->getId() ?>][attributeEncode]"
                  value="<?php echo base64_encode($item->getAttribute()) ?>"/>
               <input type="hidden" name="quote[<?php echo $item->getId() ?>][product]"
                  value="<?php echo $product->getId() ?>"/>
               <input type="hidden" name="quote[<?php echo $item->getId() ?>][quoteid]"
                  value="<?php echo $item->getId() ?>"/>
            </td>
         </tr>
         <?php
            $i++;
            endforeach;
            ?>
         <tr>
            <td colspan="6" style="text-align:right;"><?php echo $this->__('Total Items :'); ?></td>
            <td><?php echo $totalqty; ?></td>
         </tr>
         <!--<tr>
            <td colspan="6" style="text-align:right;"><?php //echo $this->__('Total Price :'); ?></td><td class="getprices"><?php //echo $totalqty; ?></td>
            </tr>-->
         <tr>
            <td colspan="6" style="text-align:right;"><?php echo $this->__('Total Amount (Excl VAT) :'); ?></td>
            <td><?php $dd =  Mage::helper('checkout/cart')->getQuote()->getSubtotal();
               echo 'R '.number_format($dd , 2)?>
            </td>
         </tr>
         <tr>
            <td colspan="6" style="text-align:right;"><?php echo $this->__('( VAT) :'); ?></td>
            <td><?php $dd1 =  (Mage::helper('checkout/cart')->getQuote()->getSubtotal() * 14)/100; 
               echo 'R '.number_format($dd1, 2)?>
            </td>
         </tr>
         <tr>
            <td colspan="6" style="text-align:right;"><?php echo $this->__('( Total Amount (Incl VAT)) :'); ?></td>
            <td><?php $quote = Mage::getModel('checkout/session')->getQuote();
               $quoteData= $quote->getData();
                $grandTotal=$quoteData['grand_total'];
                echo 'R '.number_format($grandTotal, 2);?></td>
         </tr>
      </tbody>
      <tfoot>
         <tr>
            <td colspan="100" class="a-right">
               <?php if($this->getContinueShoppingUrl()):
                  $this->getCustomerSession()->setContinueShoppingUrl($this->getContinueShoppingUrl());
                  ?>
               <button class="button btn-continue" onclick="saveProductComment('url/continue');" type="button"><span><span><?php echo $this->__('Continue Shopping') ?></span></span></button>
               <?php endif; ?>
               <?php $action = "setLocation('" . $this->getUrl('qquoteadv/index/switch2Order') . "');";
                  if (count($productNames) > 0) {
                      $action = 'initMsg();';
                  }
                  ?>
               <?php if ($this->getShowOrderReferences()) : ?>
               <button type="button" onclick="<?php echo $action; ?>"
                  title="<?php echo $this->__('Move to Shopping cart') ?>"
                  class="button btn-update">
               <span><span><?php echo $this->__('Move to Shopping cart') ?></span></span></button>
               <?php endif; ?>
               <?php $action = "setLocation('" . $this->getUrl('qquoteadv/index/clearQuote') . "');"; ?>
               <!--<button type="button" onclick="<?php echo $action; ?>" title="<?php echo $this->__('Clear Quote') ?>"
                  class="button" ><span><span><?php echo $this->__('Clear Quote') ?></span></span></button>-->
            </td>
         </tr>
      </tfoot>
   </table>
   <script type="text/javascript">decorateTable('shopping-cart-table')</script>
   <?php echo $this->getChildHtml('qquote.address'); ?>
   <div style="float:right;">
      <input type='hidden' id='customer_isQuote' name='customer[is_quote]' value='1'/>
      <input style="display:none;" type='submit' name='submitOrder' id="submitOrder" class='form-button'
         value="<?php echo $this->__('Request quote') ?>"/>
      <button onclick="$('submitOrder').click(); event.preventDefault();  event.stopPropagation();"
         class="button btn-proceed-checkout btn-checkout"
         title="<?php echo $this->__('Request quote') ?>" type="button">
      <span><span><?php echo $this->__('Request quote') ?></span></span>
      </button>
   </div>
   </div>
</form>
<script type="text/javascript">
   //<![CDATA[
   var quotelistForm = new VarienForm('quotelist');
   new RegionUpdater('country', 'region', 'region_id', <?php echo $this->helper('directory')->getRegionJson() ?>);
   $('country').addClassName('w224');
   new RegionUpdater('shipping_country', 'shipping_region', 'shipping_region_id', <?php echo $this->helper('directory')->getRegionJson() ?>);
   $('shipping_country').addClassName('w224');
   
   function requestShippingProposal() {
       var elmEmail = $('customer:email');
       if (elmEmail.value) {
           // Send quotelist form to another action
           var quoteShipForm = document.getElementById('quotelist');
           // Reset isQuote to 2
           var isQuote = document.getElementById('customer_isQuote');
           isQuote.value = '2';
           quoteShipForm.action = '<?php echo $this->getUrl('qquoteadv/index/quoteShippingEstimate'); ?>';
           quoteShipForm.submit();
   
       }
   }
   
   //]]>
</script>
<?php
   //#show pop-up window
       if (count($productNames) > 0) {
           echo $this->getLayout()->createBlock("core/template")->setTemplate('qquoteadv/quote_lightbox.phtml')->setProductNames($productNames)->toHtml();
       }
       ?>
<script language="javascript">
   var pool = new Array();
   
   function addNewLine(itemId, inputName) {
       if (!pool[itemId]) {
           pool[itemId] = 1;
       }
   
       index = pool[itemId];
       index++;
       pool[itemId] = index;
   
       var parentElemt = document.getElementById('qdiv_' + itemId);
       var childElem = document.createElement('div');
       childElem.setAttribute("id", 'div_' + itemId + '_' + index);
       parentDiv = 'div_' + itemId + '_' + index;
       inputField = 'quote_' + itemId + '_' + index;
       link = '&nbsp;&nbsp;<a style="text-decoration:none;" href="#"  onClick="removeElement(\'' + parentDiv + '\', \'' + inputField + '\'); $(\'' + parentDiv + '\').hide()"><img style="vertical-align: bottom;bottom;width:19px;height:18px;" src="<?php echo $this->getSkinUrl('images/minus-icon.png')?>"></a>';
   
       childElem.innerHTML = '<input size="6" type="text" name="' + inputName + '"  id="quote_' + itemId + '_' + index + '" value=""  class="required-entry validate-zero-or-greater required-entry input-text m5">' + link;
   
       parentElemt.appendChild(childElem);
   }
   
   function removeElement(parentElemt, childElemt) {
       var parentDiv = document.getElementById(parentElemt);
       var childElemt = document.getElementById(childElemt);
       parentDiv.removeChild(childElemt);
   }
   function saveProductComment(action){
   var quoteForm = document.getElementById('quotelist');
   quoteForm.action = '<?php echo $this->getUrl('qquoteadv/index/storeQuote/'); ?>' + action;
   quoteForm.submit();
   }
</script>
<?php endif; ?>
<script type ="text/javascript">
   jQuery(document).ready(function() {
      jQuery(".getprices").html(jQuery(".subtotal .price").html());
   });
   
</script>
