import React, { useState, useEffect, useRef } from 'react';
import { Send, Home, MapPin, DollarSign, Phone, CheckCircle, ArrowRight, MessageSquare, Building2, TrendingUp, Search } from 'lucide-react';
 
// Sample property database
const SAMPLE_PROPERTIES = [
  {
    id: 'P101',
    type: 'Plot',
    size: '150 sqyds',
    facing: 'East',
    price: '75L',
    road: '30ft',
    location: 'Kukatpally',
    address: 'Near KPHB Colony',
    premium: false
  },
  {
    id: 'P102',
    type: 'House',
    size: '200 sqyds',
    facing: 'North',
    price: '80L',
    road: '40ft',
    location: 'Miyapur',
    address: 'BHEL Township Area',
    premium: true
  },
  {
    id: 'P103',
    type: 'Flat',
    size: '1200 sqft',
    facing: 'South',
    price: '65L',
    road: '60ft',
    location: 'Gachibowli',
    address: 'Near DLF Cyber City',
    premium: true
  },
  {
    id: 'P104',
    type: 'Plot',
    size: '120 sqyds',
    facing: 'West',
    price: '45L',
    road: '30ft',
    location: 'Bachupally',
    address: 'Nizampet Road',
    premium: false
  },
  {
    id: 'P105',
    type: 'House',
    size: '180 sqyds',
    facing: 'East',
    price: '95L',
    road: '40ft',
    location: 'Manikonda',
    address: 'Near OU Colony',
    premium: true
  }
];
 
const InstantPropertyAI = () => {
  const [messages, setMessages] = useState([
    {
      role: 'assistant',
      content: `🏡 Namaste! Welcome to Instant Property AI!\n\nI'm your smart real estate assistant. How can I help you today?\n\n✅ Buy/Rent Property\n✅ Sell/List Property\n✅ Schedule Site Visit\n✅ Browse Listings\n\nType your requirement or choose an option below!`,
      timestamp: new Date()
    }
  ]);
  const [inputMessage, setInputMessage] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [userContext, setUserContext] = useState({});
  const [showPropertyModal, setShowPropertyModal] = useState(false);
  const [selectedProperty, setSelectedProperty] = useState(null);
  const messagesEndRef = useRef(null);
  const chatContainerRef = useRef(null);
 
  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };
 
  useEffect(() => {
    scrollToBottom();
  }, [messages]);
 
  const searchProperties = (location, budget, type) => {
    let results = SAMPLE_PROPERTIES;
    
    if (location) {
      results = results.filter(p => 
        p.location.toLowerCase().includes(location.toLowerCase()) ||
        p.address.toLowerCase().includes(location.toLowerCase())
      );
    }
    
    if (type) {
      results = results.filter(p => 
        p.type.toLowerCase().includes(type.toLowerCase())
      );
    }
    
    if (budget) {
      const budgetNum = parseInt(budget.replace(/[^0-9]/g, ''));
      results = results.filter(p => {
        const priceNum = parseInt(p.price.replace(/[^0-9]/g, ''));
        return priceNum <= budgetNum * 1.2; // 20% flexibility
      });
    }
    
    return results.slice(0, 5);
  };
 
  const formatPropertyList = (properties) => {
    if (properties.length === 0) {
      return "Sorry, no properties found matching your criteria. Let me suggest similar options or try different search parameters.";
    }
    
    let response = `🏘️ Here are the best properties for you:\n\n`;
    
    properties.forEach((prop, index) => {
      response += `${index + 1}) 🏷️ ID: ${prop.id} ${prop.premium ? '⭐' : ''}\n`;
      response += `   Type: ${prop.type}\n`;
      response += `   Size: ${prop.size}\n`;
      response += `   Facing: ${prop.facing}\n`;
      response += `   Price: ₹${prop.price}\n`;
      response += `   Road: ${prop.road}\n`;
      response += `   Location: ${prop.location}\n\n`;
    });
    
    response += `💡 Type "DETAILS ${properties[0].id}" for full details\n`;
    response += `📅 Type "VISIT" to schedule site visit\n`;
    response += `🔍 Type "MORE" for more options`;
    
    return response;
  };
 
  const handleSendMessage = async () => {
    if (!inputMessage.trim()) return;
 
    const userMessage = {
      role: 'user',
      content: inputMessage,
      timestamp: new Date()
    };
 
    setMessages(prev => [...prev, userMessage]);
    setInputMessage('');
    setIsLoading(true);
 
    try {
      // Handle special commands
      const lowerInput = inputMessage.toLowerCase();
      
      // Property details command
      if (lowerInput.startsWith('details ')) {
        const propertyId = inputMessage.split(' ')[1].toUpperCase();
        const property = SAMPLE_PROPERTIES.find(p => p.id === propertyId);
        
        if (property) {
          setSelectedProperty(property);
          setShowPropertyModal(true);
          
          const response = {
            role: 'assistant',
            content: `✅ Showing detailed information for Property ${propertyId}\n\nWould you like to:\n📅 Schedule a visit\n📞 Contact our team\n💬 Continue on WhatsApp`,
            timestamp: new Date()
          };
          setMessages(prev => [...prev, response]);
          setIsLoading(false);
          return;
        }
      }
      
      // Simple property search
      if (lowerInput.includes('kukatpally') || lowerInput.includes('miyapur') || lowerInput.includes('gachibowli')) {
        const properties = searchProperties(inputMessage, null, null);
        const response = {
          role: 'assistant',
          content: formatPropertyList(properties),
          timestamp: new Date()
        };
        setMessages(prev => [...prev, response]);
        setIsLoading(false);
        return;
      }
 
      // Call Claude API for intelligent responses
      const systemPrompt = `You are "Instant Property AI Agent" – a smart real estate assistant for a website in Hyderabad, India.
 
CONTEXT: User is interacting with a real estate platform to BUY, SELL, or RENT properties.
 
YOUR PERSONALITY:
- Friendly, helpful, and professional
- Use mix of Telugu words with English (e.g., "Chala baagundi!", "Super!", "Okay")
- Keep responses SHORT and CONVERSATIONAL
- Always guide step-by-step
- End with a clear question or action
 
USER FLOWS:
 
🟢 BUYER FLOW:
If user wants to buy/rent, collect:
1. Location/Area (e.g., Kukatpally, Miyapur, Gachibowli)
2. Budget (e.g., 50L, 1Cr)
3. Property type (Plot/House/Flat)
4. Size preference (optional)
 
Then search and show properties using this format:
"Here are properties in [Area]:
[List 3-5 properties with ID, type, size, price]
Type 'DETAILS P101' for full info"
 
🔵 SELLER FLOW:
If user wants to sell/list property, collect step-by-step:
1. Owner Name
2. Phone Number
3. Owner or Broker?
4. Property Type (Plot/House/Flat)
5. Sale or Rent?
6. Location
7. Address
8. Price
9. Size & Facing
10. Photos (ask them to upload)
 
After collecting all info:
"✅ Your property details are ready!
💰 Listing Fee:
₹99 - Basic Listing
₹199 - Premium Listing (top visibility)
Click PAY NOW to publish"
 
🟡 SITE VISIT:
If user wants to visit, collect:
- Name
- Phone
- Preferred date
Then confirm: "✅ Visit request submitted!"
 
🔴 PROPERTY DETAILS:
When user types "DETAILS P101":
Show full property information and ask if they want to schedule visit.
 
RULES:
- Keep responses under 200 words
- Never show owner phone number directly
- Always try to capture lead (name + phone)
- Use emojis strategically
- Be conversational, not robotic
- Guide them to next action
 
CURRENT CONVERSATION CONTEXT:
${JSON.stringify(userContext)}
 
Previous messages for context:
${messages.slice(-4).map(m => `${m.role}: ${m.content}`).join('\n')}
 
Respond naturally to: "${inputMessage}"`;
 
      const response = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514",
          max_tokens: 1000,
          system: systemPrompt,
          messages: [
            { role: "user", content: inputMessage }
          ],
        })
      });
 
      const data = await response.json();
      const assistantMessage = data.content
        .filter(item => item.type === "text")
        .map(item => item.text)
        .join("\n");
 
      const aiResponse = {
        role: 'assistant',
        content: assistantMessage,
        timestamp: new Date()
      };
 
      setMessages(prev => [...prev, aiResponse]);
      
      // Update context based on conversation
      if (lowerInput.includes('buy') || lowerInput.includes('rent') || lowerInput.includes('property')) {
        setUserContext(prev => ({ ...prev, intent: 'buyer' }));
      } else if (lowerInput.includes('sell') || lowerInput.includes('list')) {
        setUserContext(prev => ({ ...prev, intent: 'seller' }));
      }
 
    } catch (error) {
      console.error('Error calling Claude API:', error);
      const errorResponse = {
        role: 'assistant',
        content: 'Sorry, I encountered an error. Please try again! 😊',
        timestamp: new Date()
      };
      setMessages(prev => [...prev, errorResponse]);
    } finally {
      setIsLoading(false);
    }
  };
 
  const handleQuickAction = (action) => {
    setInputMessage(action);
    setTimeout(() => handleSendMessage(), 100);
  };
 
  const PropertyCard = ({ property }) => (
    <div className="bg-white rounded-lg border border-amber-100 p-4 hover:shadow-lg transition-shadow cursor-pointer"
         onClick={() => {
           setSelectedProperty(property);
           setShowPropertyModal(true);
         }}>
      <div className="flex justify-between items-start mb-3">
        <div className="flex items-center gap-2">
          <span className="text-2xl">{property.type === 'Plot' ? '📐' : property.type === 'House' ? '🏠' : '🏢'}</span>
          <div>
            <h3 className="font-semibold text-gray-900">{property.type} - {property.id}</h3>
            <p className="text-sm text-gray-600">{property.location}</p>
          </div>
        </div>
        {property.premium && (
          <span className="bg-amber-500 text-white text-xs px-2 py-1 rounded-full">⭐ Premium</span>
        )}
      </div>
      
      <div className="grid grid-cols-2 gap-2 text-sm mb-3">
        <div className="flex items-center gap-1 text-gray-700">
          <span>📏</span> {property.size}
        </div>
        <div className="flex items-center gap-1 text-gray-700">
          <span>🧭</span> {property.facing}
        </div>
        <div className="flex items-center gap-1 text-gray-700">
          <span>🛣️</span> {property.road}
        </div>
        <div className="flex items-center gap-1 font-semibold text-amber-600">
          <span>💰</span> ₹{property.price}
        </div>
      </div>
      
      <button className="w-full bg-gradient-to-r from-amber-500 to-orange-500 text-white py-2 rounded-lg hover:shadow-md transition-all text-sm font-medium">
        View Full Details
      </button>
    </div>
  );
 
  return (
    <div className="min-h-screen bg-gradient-to-br from-orange-50 via-amber-50 to-yellow-50">
      {/* Header */}
      <header className="bg-white border-b border-amber-100 shadow-sm sticky top-0 z-50">
        <div className="max-w-7xl mx-auto px-4 py-4 flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="bg-gradient-to-br from-amber-500 to-orange-600 p-2 rounded-xl">
              <Building2 className="w-6 h-6 text-white" />
            </div>
            <div>
              <h1 className="text-xl font-bold bg-gradient-to-r from-amber-600 to-orange-600 bg-clip-text text-transparent">
                Instant Property AI
              </h1>
              <p className="text-xs text-gray-600">Smart Real Estate Assistant</p>
            </div>
          </div>
          
          <div className="flex items-center gap-3">
            <button className="hidden md:flex items-center gap-2 px-4 py-2 bg-green-500 text-white rounded-lg hover:bg-green-600 transition-colors text-sm font-medium">
              <Phone className="w-4 h-4" />
              WhatsApp
            </button>
            <button className="flex items-center gap-2 px-4 py-2 border border-amber-300 text-amber-700 rounded-lg hover:bg-amber-50 transition-colors text-sm font-medium">
              <TrendingUp className="w-4 h-4" />
              List Property
            </button>
          </div>
        </div>
      </header>
 
      {/* Main Content */}
      <div className="max-w-7xl mx-auto px-4 py-8">
        <div className="grid lg:grid-cols-3 gap-6">
          {/* Left Panel - Featured Properties */}
          <div className="lg:col-span-1 space-y-4">
            <div className="bg-white rounded-xl p-5 border border-amber-100 shadow-sm">
              <h2 className="text-lg font-bold text-gray-900 mb-4 flex items-center gap-2">
                <span className="text-amber-500">⭐</span>
                Featured Properties
              </h2>
              <div className="space-y-3">
                {SAMPLE_PROPERTIES.filter(p => p.premium).slice(0, 3).map(property => (
                  <PropertyCard key={property.id} property={property} />
                ))}
              </div>
            </div>
 
            {/* Quick Stats */}
            <div className="bg-gradient-to-br from-amber-500 to-orange-600 rounded-xl p-5 text-white shadow-lg">
              <h3 className="font-bold mb-3">Platform Stats</h3>
              <div className="space-y-2 text-sm">
                <div className="flex justify-between">
                  <span>Active Listings</span>
                  <span className="font-semibold">150+</span>
                </div>
                <div className="flex justify-between">
                  <span>Happy Customers</span>
                  <span className="font-semibold">500+</span>
                </div>
                <div className="flex justify-between">
                  <span>Areas Covered</span>
                  <span className="font-semibold">25+</span>
                </div>
              </div>
            </div>
          </div>
 
          {/* Center Panel - Chat Interface */}
          <div className="lg:col-span-2">
            <div className="bg-white rounded-xl shadow-lg border border-amber-100 overflow-hidden">
              {/* Chat Header */}
              <div className="bg-gradient-to-r from-amber-500 to-orange-600 p-4 flex items-center gap-3">
                <div className="w-10 h-10 bg-white rounded-full flex items-center justify-center">
                  <MessageSquare className="w-5 h-5 text-amber-600" />
                </div>
                <div className="flex-1">
                  <h2 className="text-white font-semibold">AI Property Assistant</h2>
                  <p className="text-amber-100 text-xs">Online • Ready to help</p>
                </div>
                <div className="w-2 h-2 bg-green-400 rounded-full animate-pulse"></div>
              </div>
 
              {/* Messages Container */}
              <div 
                ref={chatContainerRef}
                className="h-[500px] overflow-y-auto p-4 space-y-4 bg-gradient-to-b from-orange-50/30 to-transparent"
              >
                {messages.map((message, index) => (
                  <div
                    key={index}
                    className={`flex ${message.role === 'user' ? 'justify-end' : 'justify-start'} animate-fadeIn`}
                  >
                    <div
                      className={`max-w-[80%] rounded-2xl px-4 py-3 ${
                        message.role === 'user'
                          ? 'bg-gradient-to-r from-amber-500 to-orange-500 text-white rounded-br-none'
                          : 'bg-white border border-amber-100 text-gray-800 rounded-bl-none shadow-sm'
                      }`}
                    >
                      <p className="text-sm whitespace-pre-line leading-relaxed">{message.content}</p>
                      <span className={`text-xs mt-1 block ${
                        message.role === 'user' ? 'text-amber-100' : 'text-gray-500'
                      }`}>
                        {message.timestamp.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit' })}
                      </span>
                    </div>
                  </div>
                ))}
                
                {isLoading && (
                  <div className="flex justify-start">
                    <div className="bg-white border border-amber-100 rounded-2xl rounded-bl-none px-4 py-3 shadow-sm">
                      <div className="flex gap-1">
                        <div className="w-2 h-2 bg-amber-500 rounded-full animate-bounce"></div>
                        <div className="w-2 h-2 bg-amber-500 rounded-full animate-bounce" style={{ animationDelay: '0.2s' }}></div>
                        <div className="w-2 h-2 bg-amber-500 rounded-full animate-bounce" style={{ animationDelay: '0.4s' }}></div>
                      </div>
                    </div>
                  </div>
                )}
                
                <div ref={messagesEndRef} />
              </div>
 
              {/* Quick Actions */}
              <div className="px-4 py-3 bg-amber-50/50 border-t border-amber-100">
                <div className="flex flex-wrap gap-2">
                  <button
                    onClick={() => handleQuickAction('I want to buy property')}
                    className="px-3 py-1.5 bg-white border border-amber-200 text-amber-700 rounded-full text-xs hover:bg-amber-50 transition-colors font-medium"
                  >
                    🏘️ Buy Property
                  </button>
                  <button
                    onClick={() => handleQuickAction('I want to sell my property')}
                    className="px-3 py-1.5 bg-white border border-amber-200 text-amber-700 rounded-full text-xs hover:bg-amber-50 transition-colors font-medium"
                  >
                    💰 Sell Property
                  </button>
                  <button
                    onClick={() => handleQuickAction('Show properties in Gachibowli')}
                    className="px-3 py-1.5 bg-white border border-amber-200 text-amber-700 rounded-full text-xs hover:bg-amber-50 transition-colors font-medium"
                  >
                    🔍 Browse Listings
                  </button>
                  <button
                    onClick={() => handleQuickAction('Schedule a site visit')}
                    className="px-3 py-1.5 bg-white border border-amber-200 text-amber-700 rounded-full text-xs hover:bg-amber-50 transition-colors font-medium"
                  >
                    📅 Site Visit
                  </button>
                </div>
              </div>
 
              {/* Input Area */}
              <div className="p-4 bg-white border-t border-amber-100">
                <div className="flex gap-2">
                  <input
                    type="text"
                    value={inputMessage}
                    onChange={(e) => setInputMessage(e.target.value)}
                    onKeyPress={(e) => e.key === 'Enter' && handleSendMessage()}
                    placeholder="Type your message... (e.g., 'Show plots in Miyapur')"
                    className="flex-1 px-4 py-3 border border-amber-200 rounded-xl focus:outline-none focus:ring-2 focus:ring-amber-500 focus:border-transparent"
                    disabled={isLoading}
                  />
                  <button
                    onClick={handleSendMessage}
                    disabled={isLoading || !inputMessage.trim()}
                    className="px-6 py-3 bg-gradient-to-r from-amber-500 to-orange-500 text-white rounded-xl hover:shadow-lg transition-all disabled:opacity-50 disabled:cursor-not-allowed font-medium"
                  >
                    <Send className="w-5 h-5" />
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
 
      {/* Property Detail Modal */}
      {showPropertyModal && selectedProperty && (
        <div className="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4" onClick={() => setShowPropertyModal(false)}>
          <div className="bg-white rounded-2xl max-w-2xl w-full max-h-[90vh] overflow-y-auto shadow-2xl" onClick={(e) => e.stopPropagation()}>
            <div className="bg-gradient-to-r from-amber-500 to-orange-600 p-6 text-white">
              <h2 className="text-2xl font-bold mb-2">Property Details</h2>
              <p className="text-amber-100">ID: {selectedProperty.id}</p>
            </div>
            
            <div className="p-6 space-y-4">
              <div className="grid grid-cols-2 gap-4">
                <div className="bg-amber-50 p-4 rounded-lg">
                  <p className="text-xs text-gray-600 mb-1">Type</p>
                  <p className="font-semibold text-gray-900">{selectedProperty.type}</p>
                </div>
                <div className="bg-amber-50 p-4 rounded-lg">
                  <p className="text-xs text-gray-600 mb-1">Price</p>
                  <p className="font-semibold text-amber-600">₹{selectedProperty.price}</p>
                </div>
                <div className="bg-amber-50 p-4 rounded-lg">
                  <p className="text-xs text-gray-600 mb-1">Size</p>
                  <p className="font-semibold text-gray-900">{selectedProperty.size}</p>
                </div>
                <div className="bg-amber-50 p-4 rounded-lg">
                  <p className="text-xs text-gray-600 mb-1">Facing</p>
                  <p className="font-semibold text-gray-900">{selectedProperty.facing}</p>
                </div>
                <div className="bg-amber-50 p-4 rounded-lg">
                  <p className="text-xs text-gray-600 mb-1">Road Width</p>
                  <p className="font-semibold text-gray-900">{selectedProperty.road}</p>
                </div>
                <div className="bg-amber-50 p-4 rounded-lg">
                  <p className="text-xs text-gray-600 mb-1">Location</p>
                  <p className="font-semibold text-gray-900">{selectedProperty.location}</p>
                </div>
              </div>
              
              <div className="bg-blue-50 p-4 rounded-lg">
                <p className="text-sm text-gray-700">
                  <MapPin className="w-4 h-4 inline mr-2 text-blue-600" />
                  {selectedProperty.address}
                </p>
              </div>
              
              <div className="flex gap-3">
                <button className="flex-1 bg-gradient-to-r from-amber-500 to-orange-500 text-white py-3 rounded-lg hover:shadow-lg transition-all font-medium">
                  📅 Schedule Visit
                </button>
                <button className="flex-1 bg-green-500 text-white py-3 rounded-lg hover:bg-green-600 transition-colors font-medium">
                  💬 WhatsApp
                </button>
              </div>
              
              <button 
                onClick={() => setShowPropertyModal(false)}
                className="w-full border border-gray-300 text-gray-700 py-3 rounded-lg hover:bg-gray-50 transition-colors"
              >
                Close
              </button>
            </div>
          </div>
        </div>
      )}
 
      {/* Footer */}
      <footer className="bg-white border-t border-amber-100 mt-12 py-8">
        <div className="max-w-7xl mx-auto px-4 text-center">
          <p className="text-gray-600 text-sm mb-2">
            🏡 Instant Property AI - Your Trusted Real Estate Partner
          </p>
          <p className="text-gray-500 text-xs">
            Platform Fee: 0.5% - 1% commission on successful deals | Listing Fee: ₹99-₹199
          </p>
        </div>
      </footer>
 
      <style jsx>{`
        @keyframes fadeIn {
          from {
            opacity: 0;
            transform: translateY(10px);
          }
          to {
            opacity: 1;
            transform: translateY(0);
          }
        }
        
        .animate-fadeIn {
          animation: fadeIn 0.3s ease-out;
        }
      `}</style>
    </div>
  );
};
 
export default InstantPropertyAI;
