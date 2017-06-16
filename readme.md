***The included source code, service and information is provided as is, and OmniUpdate makes no promises or guarantees about its use or misuse. The source code provided is recommended for advanced users and may not be compatible with all implementations of OU Campus.***

# LDP Galleries

When a user adds an LDP gallery to a page, the page is populated with XML information beginning with a node titled `gallery`. This node will contain all the information about the pictures within the gallery, which includes, URL, Caption, Title, Description, Thumbnail URL, Thumbnail Size (width and height).
In version 10, 'link' is also introduced, which is used to make an image in a slider link to another web page. In previous implementations, the 'caption'  was used for this instead.
The usage of title, description, caption, and link vary slightly depending on the gallery type. Some gallery types do not use all of these items.

Shown below is an example of the XML that is output to the page, with extra information omitted.
    
    <gallery asset_id="4800" ... >
    
        ...
        
        <images>
           
            <image ... url="http://cspooner.oudemo.com/v10/ldpgalleries/.private_ldp/a4800/staging/master/01ffeac5-3c93-4935-a847-f29f83836fb7.jpg" ...>
                ... 
                <thumbnail ... url="http://cspooner.oudemo.com/v10/ldpgalleries/.private_ldp/a4800/staging/thumb/01ffeac5-3c93-4935-a847-f29f83836fb7.jpg">
                    <width>100</width>
                    <height>100</height>
                </thumbnail>
                <title>test</title>
                <description>test</description>
                <caption>test</caption>
                <link>http://www.google.com</link>
            </image>
            
            <image ... url="http://cspooner.oudemo.com/v10/ldpgalleries/.private_ldp/a4800/staging/master/6d9a48b0-347d-4fd1-95d0-fc4d077418c9.jpg" ...>
                ...
                <thumbnail ... url="http://cspooner.oudemo.com/v10/ldpgalleries/.private_ldp/a4800/staging/thumb/6d9a48b0-347d-4fd1-95d0-fc4d077418c9.jpg">
                    <width>100</width>
                    <height>100</height>
                </thumbnail>
                <title>test2</title>
                <description>test2</description>
                <caption>test2</caption>
                <link>http://www.google.com</link>
            </image>

        </images>

    </gallery>


## PCF and TMPL Files

For most implementations, the user will be able to select a gallery type in Page Properties. This gallery type will apply for all gallery assets on the page. However, a user could add the same asset on another page with a different type selected.

    <parameter section="LDP Gallery" name="gallery-type" type="select" group="Everyone" prompt="Gallery Type" alt="Select the output type for gallery assets on this page.">
        <option value="bx-slider" selected="true">BX Slider</option>
        <option value="flex-slider" selected="false">Flex Slider</option>
        <option value="magnific-popup" selected="false">Magnific PopUp</option>
        <option value="pretty-photo" selected="false">Pretty Photo</option>
        <option value="fancy-box" selected="false">Fancy Box</option>
    </parameter>

## galleries.xsl

The template matches in galleries.xsl will then match the gallery node and, depending on the type selected by the user, apply a different transformation. Note that the parameter galleryType is the value of the selected page property. If you do not have pcf-params, simply redefine this parameter to the xpath of the selected option.
	
    <xsl:template match="gallery">
    <xsl:param name="galleryType" select="ou:pcfparam('galleryType')" />
    <xsl:variable name="galleryId" select="@asset_id"/>

        <xsl:choose>
        	
			<xsl:when test="$gallery-type='bx-slider'">
				<ul class="bx-slider" style="margin:0;padding:0;">
					
				<xsl:for-each select="images/image">
						<li>
							<img src="{@url}" title="{title}" alt="{description}" style="width:100%; height: auto;"/>
						</li>
				</xsl:for-each>
					
				</ul>
			</xsl:when>
            
            ...
        	
			<xsl:when test="$gallery-type='pretty-photo'">
				
				<ul class="thumbnails">
					<xsl:for-each select="images/image">
						<li>
							<a href="{@url}" rel="prettyPhoto[{$gallery-id}]" title="{title}"  class="thumbnail">
								<img src="{thumbnail/@url}" alt="{title}" style="height:{thumbnail/height}px; width:{thumbnail/width}px;"/>
							</a>
						</li>
					</xsl:for-each>	
				</ul>
				
			</xsl:when>	
        	
        	...
    	
    	</xsl:choose>
    	
    </xsl:template>
	
Since most implementations will include only 1 slider gallery and 1 thumbnail gallery, simply remove the extra conditions within the `xsl:choose`. Additionally, new conditionals can be added if the implementation has a custom gallery type.	You may also want to specify a fall-back gallery type as `xsl:otherwise`.	

Additionally the function `ou:gallery-headcode` is used by common.xsl to copy any needed resource files. It will also be necessary to remove extra conditionals. If an implementation has a custom gallery type, it is not usually necessary to add it to this function as the resource scripts are usually already part of the design templates.

Note the use of {$domain}. This can be used to define the url if the staging root is different than the production. It is optional.

    <xsl:function name="ou:gallery-headcode">
    <xsl:param name="galleryType" />
        
        <xsl:choose>
            
			<xsl:when test="$gallery-type='bx-slider'">
				
				<script src="{$domain}/_resources/ldp/galleries/bx-slider/jquery.bxslider.min.js"></script>
				<link href="{$domain}/_resources/ldp/galleries/bx-slider/jquery.bxslider.css" rel="stylesheet" />
				
				<script type="text/javascript">
					$(document).ready(function(){
						$('.bx-slider').bxSlider({
							mode: 'fade',
							captions: true
						});
					});	
				</script>
				
			</xsl:when>
				
			<xsl:when test="$gallery-type='pretty-photo'">			
				
				<link rel="stylesheet" href="{$domain}/_resources/ldp/galleries/bootstrap-thumbnails.css"/> 			
				<link rel="stylesheet" href="{$domain}/_resources/ldp/galleries/pretty-photo/prettyPhoto.css" type="text/css" media="screen" title="prettyPhoto main stylesheet" charset="utf-8" />
				
				<script src="{$domain}/_resources/ldp/galleries/pretty-photo/jquery.prettyPhoto.js" type="text/javascript" charset="utf-8"></script>
				<script type="text/javascript">
					$(document).ready(function() {
						$("a[rel^='prettyPhoto']").prettyPhoto();
					});
				</script>
				
			</xsl:when>
        
        </xsl:choose>
    
    </xsl:function>
    
## common.xsl

In common.xsl, you will want to call the gallery headcode and footcode functions. This is usually done within the common-headcode and common-footcode templates. Ensure that the footcode is called after jquery is loaded on the page. Include them as follows:

    <xsl:if test="descendant::gallery"> 
    	<xsl:copy-of select="ou:gallery-headcode($gallery-type)"/>
    </xsl:if>	

## Uploading the Resource Files

Simply upload the folders for each gallery type into the _resources directory and publish. If you run into difficulty, check the paths of the files in the `ou:gallery-headcode` function. Additionally, thumbnail gallery types will require bootstrap-thumbnails.css.

## Troubleshooting

### Undefined jQuery Function

Sometimes when newer versions of jQuery are introduced, the functions used by the gallery javaScript no longer exist. If that is the case, then jQuery migrate can be added, which will contain the missing functions.

    <script src="http://code.jquery.com/jquery-migrate-1.1.0.min.js"></script>

### Conflicts with jQuery and other JavaScript libraries

If there is a conflict between jQuery and another library, noConflict mode can be used. See http://learn.jquery.com/using-jquery-core/avoid-conflicts-other-libraries/ for more information.

