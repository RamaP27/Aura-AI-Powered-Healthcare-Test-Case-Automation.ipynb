# Aura-AI-Powered-Healthcare-Test-Case-Automation.ipynb
Aura: AI-Powered Healthcare Test Case Automation
import json
import re
from datetime import datetime
from typing import Dict, List, Optional
from dataclasses import dataclass, asdict
import hashlib

# Mock AI model for demonstration purposes
class MockGeminiModel:
    """Mock Gemini model that simulates AI-powered test case generation"""
    
    def __init__(self):
        self.healthcare_keywords = [
            'patient', 'medical', 'device', 'diagnosis', 'treatment', 'prescription',
            'vital signs', 'EHR', 'EMR', 'FDA', 'HIPAA', 'compliance', 'safety',
            'authentication', 'authorization', 'data integrity', 'audit trail'
        ]
        
        self.regulatory_standards = {
            'FDA': 'FDA 21 CFR Part 820',
            'IEC': 'IEC 62304',
            'ISO': 'ISO 13485',
            'HIPAA': 'HIPAA Security Rule'
        }
    
    def extract_entities(self, text: str) -> Dict[str, List[str]]:
        """Extract healthcare-specific entities from requirements text"""
        entities = {
            'keywords': [],
            'regulatory_refs': [],
            'actions': [],
            'data_elements': []
        }
        
        text_lower = text.lower()
        
        # Extract healthcare keywords
        for keyword in self.healthcare_keywords:
            if keyword.lower() in text_lower:
                entities['keywords'].append(keyword)
        
        # Extract regulatory references
        for reg, full_name in self.regulatory_standards.items():
            if reg.lower() in text_lower:
                entities['regulatory_refs'].append(full_name)
        
        # Extract action verbs (simplified)
        action_patterns = r'\b(validate|verify|authenticate|authorize|store|retrieve|display|calculate|process|generate|send|receive)\b'
        entities['actions'] = list(set(re.findall(action_patterns, text_lower)))
        
        # Extract data elements (simplified)
        data_patterns = r'\b(password|username|patient ID|medical record|vital signs|temperature|blood pressure|medication)\b'
        entities['data_elements'] = list(set(re.findall(data_patterns, text_lower, re.IGNORECASE)))
        
        return entities
    
    def generate_test_cases(self, requirement: str, entities: Dict) -> List[Dict]:
        """Generate test cases based on requirement and extracted entities"""
        test_cases = []
        
        # Generate different types of test cases based on content
        if any('authenticate' in action for action in entities['actions']):
            test_cases.append(self._generate_authentication_test_case(requirement, entities))
        
        if any('patient' in keyword for keyword in entities['keywords']):
            test_cases.append(self._generate_patient_data_test_case(requirement, entities))
        
        if entities['regulatory_refs']:
            test_cases.append(self._generate_compliance_test_case(requirement, entities))
        
        # Default functional test case
        if not test_cases:
            test_cases.append(self._generate_default_test_case(requirement, entities))
        
        return test_cases
    
    def _generate_authentication_test_case(self, requirement: str, entities: Dict) -> Dict:
        return {
            'title': 'User Authentication Validation',
            'priority': 'High',
            'type': 'Security',
            'steps': [
                {'step': 1, 'action': 'Navigate to login page', 'expected': 'Login page displays correctly'},
                {'step': 2, 'action': 'Enter valid credentials', 'expected': 'User is authenticated successfully'},
                {'step': 3, 'action': 'Enter invalid credentials', 'expected': 'Authentication fails with appropriate error message'},
                {'step': 4, 'action': 'Verify session timeout', 'expected': 'User is logged out after configured timeout period'}
            ],
            'test_data': ['Valid username/password pairs', 'Invalid credentials', 'Session timeout configuration'],
            'regulatory_compliance': entities.get('regulatory_refs', []),
            'risk_level': 'High'
        }
    
    def _generate_patient_data_test_case(self, requirement: str, entities: Dict) -> Dict:
        return {
            'title': 'Patient Data Management Validation',
            'priority': 'Critical',
            'type': 'Functional',
            'steps': [
                {'step': 1, 'action': 'Access patient record system', 'expected': 'System loads patient dashboard'},
                {'step': 2, 'action': 'Search for patient by ID', 'expected': 'Correct patient record is retrieved'},
                {'step': 3, 'action': 'Verify data integrity', 'expected': 'All patient data fields are accurate and complete'},
                {'step': 4, 'action': 'Test data access controls', 'expected': 'Only authorized users can access patient data'}
            ],
            'test_data': ['Sample patient IDs', 'Patient demographic data', 'Medical history records'],
            'regulatory_compliance': ['HIPAA Security Rule', 'FDA 21 CFR Part 820'],
            'risk_level': 'Critical'
        }
    
    def _generate_compliance_test_case(self, requirement: str, entities: Dict) -> Dict:
        return {
            'title': 'Regulatory Compliance Validation',
            'priority': 'High',
            'type': 'Compliance',
            'steps': [
                {'step': 1, 'action': 'Review audit trail functionality', 'expected': 'All user actions are logged'},
                {'step': 2, 'action': 'Verify data encryption', 'expected': 'Sensitive data is encrypted at rest and in transit'},
                {'step': 3, 'action': 'Test backup and recovery', 'expected': 'Data can be restored from backups'},
                {'step': 4, 'action': 'Validate user access controls', 'expected': 'Role-based access is properly enforced'}
            ],
            'test_data': ['User roles and permissions', 'Audit log samples', 'Encryption keys'],
            'regulatory_compliance': entities.get('regulatory_refs', []),
            'risk_level': 'High'
        }
    
    def _generate_default_test_case(self, requirement: str, entities: Dict) -> Dict:
        return {
            'title': 'Functional Requirement Validation',
            'priority': 'Medium',
            'type': 'Functional',
            'steps': [
                {'step': 1, 'action': 'Verify system functionality', 'expected': 'System behaves as specified'},
                {'step': 2, 'action': 'Test positive scenarios', 'expected': 'All valid inputs produce expected results'},
                {'step': 3, 'action': 'Test negative scenarios', 'expected': 'Invalid inputs are handled gracefully'},
                {'step': 4, 'action': 'Verify error handling', 'expected': 'Appropriate error messages are displayed'}
            ],
            'test_data': ['Valid input data', 'Invalid input data', 'Edge cases'],
            'regulatory_compliance': entities.get('regulatory_refs', []),
            'risk_level': 'Medium'
        }

@dataclass
class TestCase:
    """Test case data structure"""
    id: str
    title: str
    priority: str
    type: str
    steps: List[Dict]
    test_data: List[str]
    regulatory_compliance: List[str]
    risk_level: str
    created_date: str
    requirement_hash: str
    traceability_id: str

class AuraTestCaseGenerator:
    """Main Aura system for automated test case generation"""
    
    def __init__(self):
        self.model = MockGeminiModel()
        self.generated_tests = []
        self.traceability_matrix = {}
    
    def process_requirement(self, requirement_text: str, source_file: str = "manual_input") -> List[TestCase]:
        """Process healthcare requirement and generate test cases"""
        print(f"🔍 Processing requirement from: {source_file}")
        print(f"📝 Requirement text: {requirement_text[:100]}...")
        
        # Step 1: Extract entities using AI
        entities = self.model.extract_entities(requirement_text)
        print(f"🎯 Extracted entities: {entities}")
        
        # Step 2: Generate test cases
        raw_test_cases = self.model.generate_test_cases(requirement_text, entities)
        
        # Step 3: Structure test cases
        structured_test_cases = []
        requirement_hash = hashlib.md5(requirement_text.encode()).hexdigest()[:8]
        
        for i, raw_case in enumerate(raw_test_cases):
            test_id = f"TC_{requirement_hash}_{i+1:03d}"
            traceability_id = f"REQ_{requirement_hash}_TEST_{i+1}"
            
            test_case = TestCase(
                id=test_id,
                title=raw_case['title'],
                priority=raw_case['priority'],
                type=raw_case['type'],
                steps=raw_case['steps'],
                test_data=raw_case['test_data'],
                regulatory_compliance=raw_case['regulatory_compliance'],
                risk_level=raw_case['risk_level'],
                created_date=datetime.now().isoformat(),
                requirement_hash=requirement_hash,
                traceability_id=traceability_id
            )
            
            structured_test_cases.append(test_case)
            
            # Update traceability matrix
            self.traceability_matrix[traceability_id] = {
                'requirement_source': source_file,
                'requirement_text': requirement_text,
                'test_case_id': test_id,
                'compliance_standards': raw_case['regulatory_compliance'],
                'created_date': test_case.created_date
            }
        
        self.generated_tests.extend(structured_test_cases)
        return structured_test_cases
    
    def export_to_jira_format(self, test_cases: List[TestCase]) -> List[Dict]:
        """Export test cases in Jira-compatible format"""
        jira_issues = []
        
        for test_case in test_cases:
            # Format steps for Jira
            steps_text = "\n".join([
                f"{step['step']}. {step['action']}\n   Expected: {step['expected']}"
                for step in test_case.steps
            ])
            
            # Create Jira issue format
            jira_issue = {
                "fields": {
                    "project": {"key": "HEALTHCARE"},
                    "summary": test_case.title,
                    "description": f"""
*Test Case ID:* {test_case.id}
*Priority:* {test_case.priority}
*Type:* {test_case.type}
*Risk Level:* {test_case.risk_level}

*Test Steps:*
{steps_text}

*Test Data Required:*
{chr(10).join(f"• {data}" for data in test_case.test_data)}

*Regulatory Compliance:*
{chr(10).join(f"• {std}" for std in test_case.regulatory_compliance)}

*Traceability ID:* {test_case.traceability_id}
""",
                    "issuetype": {"name": "Test"},
                    "priority": {"name": test_case.priority},
                    "labels": [test_case.type.lower(), "automated-generation"] + 
                             [std.lower().replace(" ", "-") for std in test_case.regulatory_compliance]
                }
            }
            jira_issues.append(jira_issue)
        
        return jira_issues
    
    def generate_traceability_report(self) -> Dict:
        """Generate comprehensive traceability report"""
        return {
            "report_generated": datetime.now().isoformat(),
            "total_requirements_processed": len(set(entry['requirement_source'] for entry in self.traceability_matrix.values())),
            "total_test_cases_generated": len(self.generated_tests),
            "traceability_matrix": self.traceability_matrix,
            "compliance_coverage": self._get_compliance_coverage(),
            "risk_distribution": self._get_risk_distribution()
        }
    
    def _get_compliance_coverage(self) -> Dict:
        """Analyze compliance standard coverage"""
        standards = {}
        for test_case in self.generated_tests:
            for standard in test_case.regulatory_compliance:
                standards[standard] = standards.get(standard, 0) + 1
        return standards
    
    def _get_risk_distribution(self) -> Dict:
        """Analyze risk level distribution"""
        risks = {}
        for test_case in self.generated_tests:
            risks[test_case.risk_level] = risks.get(test_case.risk_level, 0) + 1
        return risks

# Demo function
def run_aura_demo():
    """Run a demonstration of the Aura system"""
    print("🚀 Welcome to Aura: AI-Powered Healthcare Test Case Automation Demo")
    print("=" * 70)
    
    # Initialize Aura system
    aura = AuraTestCaseGenerator()
    
    # Sample healthcare requirements
    sample_requirements = [
        {
            "source": "Patient_Authentication_Spec.pdf",
            "text": "The system must authenticate healthcare providers using secure credentials before allowing access to patient medical records. Authentication must comply with HIPAA security requirements and maintain an audit trail of all access attempts."
        },
        {
            "source": "Vital_Signs_Monitor_Req.docx", 
            "text": "The medical device software shall continuously monitor patient vital signs including heart rate, blood pressure, and temperature. The system must validate all measurements against FDA approved ranges and alert medical staff when values exceed safe thresholds."
        },
        {
            "source": "EHR_Data_Management.xml",
            "text": "Electronic Health Record system must securely store and retrieve patient data with full data integrity verification. The system shall implement role-based access control and comply with ISO 13485 standards for medical device software."
        }
    ]
    
    # Process each requirement
    all_test_cases = []
    for req in sample_requirements:
        print(f"\n📋 Processing: {req['source']}")
        print("-" * 50)
        
        test_cases = aura.process_requirement(req['text'], req['source'])
        all_test_cases.extend(test_cases)
        
        print(f"✅ Generated {len(test_cases)} test case(s)")
        
        # Display generated test cases
        for tc in test_cases:
            print(f"\n🧪 Test Case: {tc.title}")
            print(f"   ID: {tc.id}")
            print(f"   Priority: {tc.priority} | Type: {tc.type} | Risk: {tc.risk_level}")
            print(f"   Compliance: {', '.join(tc.regulatory_compliance)}")
            print(f"   Steps: {len(tc.steps)} test steps defined")
    
    # Export to Jira format
    print(f"\n📤 Exporting to Jira format...")
    jira_issues = aura.export_to_jira_format(all_test_cases)
    print(f"✅ Exported {len(jira_issues)} Jira issues")
    
    # Generate traceability report
    print(f"\n📊 Generating traceability report...")
    report = aura.generate_traceability_report()
    
    print(f"\n📈 AURA AUTOMATION SUMMARY")
    print("=" * 70)
    print(f"Requirements Processed: {report['total_requirements_processed']}")
    print(f"Test Cases Generated: {report['total_test_cases_generated']}")
    print(f"Compliance Coverage: {report['compliance_coverage']}")
    print(f"Risk Distribution: {report['risk_distribution']}")
    
    # Sample output - show first test case in detail
    if all_test_cases:
        print(f"\n🔍 SAMPLE TEST CASE DETAILS")
        print("=" * 70)
        sample_tc = all_test_cases[0]
        print(f"Title: {sample_tc.title}")
        print(f"ID: {sample_tc.id}")
        print(f"Traceability: {sample_tc.traceability_id}")
        print(f"Test Steps:")
        for step in sample_tc.steps:
            print(f"  {step['step']}. {step['action']}")
            print(f"     Expected: {step['expected']}")
    
    # Sample Jira export
    if jira_issues:
        print(f"\n🎫 SAMPLE JIRA EXPORT")
        print("=" * 70)
        sample_jira = jira_issues[0]
        print(f"Summary: {sample_jira['fields']['summary']}")
        print(f"Labels: {sample_jira['fields']['labels']}")
        print("Description preview:")
        print(sample_jira['fields']['description'][:200] + "...")
    
    print(f"\n🎉 Demo completed successfully!")
    print("💡 In production, this would integrate with:")
    print("   • Google Cloud Vertex AI for advanced NLP")
    print("   • Real Jira/Polarion APIs for automatic ticket creation")
    print("   • BigQuery for compliance reporting and analytics")
    print("   • Document AI for processing complex PDF requirements")
    
    return aura, report

if __name__ == "__main__":
    # Run the demo
    aura_system, final_report = run_aura_demo()
    
    # Optional: Save results to JSON files
    print(f"\n💾 Saving demo results...")
    
    with open('aura_test_cases.json', 'w') as f:
        test_cases_data = [asdict(tc) for tc in aura_system.generated_tests]
        json.dump(test_cases_data, f, indent=2)
    
    with open('aura_traceability_report.json', 'w') as f:
        json.dump(final_report, f, indent=2)
    
    print("✅ Demo results saved to JSON files")
    print("🔗 Files created: aura_test_cases.json, aura_traceability_report.json")
